---
title: Write an OpenAEV database migration
description: Add a Flyway migration to OpenAEV that changes the PostgreSQL schema or its data safely, name it so it runs in order, test it, and pass the migrations guard of the CI.
doc_type: how-to
products:
  - openaev
personas: [developer]
owner: docs-platform
last_reviewed: 2026-11-09
sidebar_position: 7
no_screenshot: "A migration is Java code that runs when the platform starts: it has no screen, and this page works in the code and a terminal."
migrated_from:
  - openaev:/development/database-migrations/
---

OpenAEV keeps its schema in PostgreSQL and changes it with migrations: Java classes that [Flyway](https://www.red-gate.com/products/flyway/) runs once, in order, when the platform starts. When your change to OpenAEV needs a new table, a new column or a data fix, you add a migration. This page creates one with the right name, writes it so that it is safe on a database in production, tests it, and gets it past the checks of the continuous integration (CI). Set up the code first: [Build and run from source](/develop/contribute/build-from-source/#build-and-run-openaev).

::diagram{id=develop-contribute-database-migrations}

## How OpenAEV runs migrations

- The migrations are Java classes in `openaev-api/src/main/java/io/openaev/migration/`, never SQL files. Each extends `BaseJavaMigration` and is a Spring `@Component`.
- At every start, Flyway compares them with the `migrations` table of the database, the history of what it applied, and runs the new ones in the order of their version: the part of the class name before `__`, read as numbers separated by `_`.
- A migration runs once. Flyway keeps a checksum of each: never edit, rename or delete a migration that is part of a release, write a new one.
- Flyway refuses by default a new migration whose version sorts before one already applied. `OPENAEV_FLYWAY_OUT_OF_ORDER=true` (`spring.flyway.out-of-order`) lets it run them anyway, which helps when you switch between branches in development. Applied migrations that no longer exist in the code are ignored (`spring.flyway.ignore-migration-patterns`, `*:missing`). The keys are in the [`spring.flyway` block](/reference/openaev/configuration/#spring-flyway).

The name has the form `V6_<yyyyMMddHHmmssSSS>__<Description>`: the major version 6, then the UTC time of its creation to the millisecond, so a new migration sorts after every existing one.

## Create the migration

:::steps
1. Build the repository once, which also installs the OpenAEV Maven plugin in your local repository:

   ```bash
   mvn clean install -DskipTests
   ```

2. From the `openaev-api` module, create the class. `reason` gives its description; characters other than letters, digits and `_` become `_`:

   ```bash
   cd openaev-api
   mvn openaev:migration -Dreason="Add collector console url"
   ```

   The plugin writes `src/main/java/io/openaev/migration/V6_<timestamp>__Add_collector_console_url.java`, with a `migrate` method that runs `SELECT 1;`.

3. Replace `SELECT 1;` with your change, following the rules below.
:::

## Write it safely

A migration runs on databases you will never see, some with millions of rows, during a rolling update where pods of the previous version still run. The rules of the repository:

| Rule | Why | How |
| --- | --- | --- |
| Idempotent | A migration interrupted halfway, or replayed on a database restored from a backup, must not fail | `CREATE TABLE IF NOT EXISTS`, `ADD COLUMN IF NOT EXISTS`, `CREATE INDEX IF NOT EXISTS`, `DROP ... IF EXISTS`; a constraint in a `DO $$ BEGIN IF NOT EXISTS (...) THEN ... END IF; END $$` block |
| A default for every new `NOT NULL` column | The table already has rows | `ADD COLUMN IF NOT EXISTS ... NOT NULL DEFAULT ...` |
| No drop without a deprecation first | Pods of the previous version still read the column during the update | Stop using it in one release, drop it in a later one |
| Tenant isolation for a table of tenant data | OpenAEV separates its tenants in the database | `tenant_id VARCHAR(255) NOT NULL`, a foreign key to `tenants(tenant_id)` with `ON DELETE CASCADE`, an index on `tenant_id`, and every unique constraint on `(field, tenant_id)` |
| Batches for large data changes | A single update locks the table for the whole run | Batches of 1,000 rows, walking the primary key; override `canExecuteInTransaction()` to return `false` so each batch commits on its own |
| A re-feed of the search index when indexed data changes | The search engine keeps a copy of what it indexes | Move the model's `indexing_status` cursor back and call `EsIndexingUtils.requestRefeedAtStartup("<model>")`, never `DELETE FROM indexing_status`. A new field on an indexed model needs nothing |

`V6_20261026151200000__Add_finding_value_display` shows a batched backfill outside a transaction, and `V6_20261022140626000__Request_finding_refeed_after_sensitive_value_masking` a re-feed of the search index.

:::example[Store the console address of each collector]
A developer adds to OpenAEV the address of the console of each collector's tool, so that the alerts of Meridian SOC, the alert store of Grand Meridian Hotels, link to `https://alerts.example.com`. The column is nullable, so the existing rows need no value:

```java
package io.openaev.migration;

import java.sql.Statement;
import org.flywaydb.core.api.migration.BaseJavaMigration;
import org.flywaydb.core.api.migration.Context;
import org.springframework.stereotype.Component;

/** Adds the address of the console of a collector's tool, to build links to its alerts. */
@Component
public class V6_20261109143015201__Add_collector_console_url extends BaseJavaMigration {

  @Override
  public void migrate(Context context) throws Exception {
    try (Statement statement = context.getConnection().createStatement()) {
      statement.execute(
          "ALTER TABLE collectors ADD COLUMN IF NOT EXISTS collector_console_url TEXT");
    }
  }
}
```

At the next start of the back end, Flyway applies it once, and `collectors` has the new column. The `Collector` entity then maps it with `@Column(name = "collector_console_url")`.
:::

## Run it

:::steps
1. Start the back end with the `dev` profile, as in [Build and run from source](/develop/contribute/build-from-source/#build-and-run-openaev):

   ```bash
   mvn spring-boot:run -pl openaev-api -Dspring-boot.run.profiles=dev
   ```

   Flyway runs the new migration before the platform serves requests. An exception in it stops the start.

2. Check that Flyway recorded it:

   ```bash
   docker exec openaev-dev-pgsql psql -U openaev -d openaev \
     -c "SELECT version, description, success FROM migrations ORDER BY installed_rank DESC LIMIT 3;"
   ```

   The first line is your migration, with `success` at `t`.
:::

## Test it

A migration that changes data gets a test in `openaev-api/src/test/java/io/openaev/migration/`. The tests of the repository extend `IntegrationTest`, inject the migration with `@Autowired`, seed rows over JDBC, call `migrate` with a `Context` that returns the test's connection, and check the rows. Run it twice in one test to prove it is idempotent. `SanitizeDocumentNamesMigrationTest` is a complete example. Run one test class with the services of the development environment started:

```bash
mvn test -pl openaev-api -Dtest=AddCollectorConsoleUrlMigrationTest
```

## Pass the migrations guard

The CI of every pull request compares the migration files of your branch with those of the last release. It fails when the block of released migrations changed: a released migration renamed or deleted, or a new migration that does not sort after all of them. It does not see edits inside a released file, which Flyway's checksum would then refuse on every database that already applied it.

Two migrations not released yet may sort in either order: the guard only checks that they come after the released ones. A development database that already applied the later one refuses the earlier one until you start the platform with `OPENAEV_FLYWAY_OUT_OF_ORDER=true`.

## What's next

- [Build and run from source](/develop/contribute/build-from-source/), to run the platform and the tests.
- [Executors](/develop/integration-model/executors/), a part of the platform you may extend with a migration.
