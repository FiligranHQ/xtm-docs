---
title: Turn on the debug mode to find slow requests
description: Turn on the debug mode of OpenAEV to trace each request with its SQL, find N+1 queries and record the JVM, safely in production, then read the output and turn it off.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
no_screenshot: "The debug mode is set in the configuration and writes files on the host: the diagram shows what it produces and where."
migrated_from:
  - openaev:/administration/debug-mode/
---

The debug mode of OpenAEV traces what the platform does, to diagnose a bug or a slow screen on your own installation. It ties the log lines of a request to a trace identifier and logs every SQL statement with its time and its masked values. It flags the requests that repeat the same query, and records the Java virtual machine (JVM) with Java Flight Recorder (JFR). It is off by default and costs nothing then. This page turns it on safely, reads what it writes, and turns it off. Raising the log level only is in [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/#raise-the-log-level).

## Before you start

- You can change the environment of the `openaev` service, restart it, and read the files it writes.
- The output folder is writable and, in a container, on a volume. On a shared platform, restrict access to it: the SQL log mixes the queries of every tenant, with their tenant identifiers in clear.
- The JFR recording with the default `profile` settings has a real cost on a busy platform. Plan a window, or use the lighter settings below.

## What it produces

::diagram{id=openaev-debug-mode-outputs}

| Output | Where | What it holds |
| --- | --- | --- |
| Trace identifier | Every console log line, as `[traceId-spanId]` | Groups the lines of one request. No tracing backend is needed |
| SQL detail | `openaev-debug-sql.log` in the output folder, rotated | Every SQL statement, with its time, its parameters masked, and the trace, tenant and user of the request |
| ORM summary | The console, or `openaev-debug-orm.log` | One line per request: its number of queries and their time, its user, and an `N+1 SUSPECTED` warning when the same `SELECT` repeats |
| JFR recording | `.jfr` files in the output folder | CPU, memory and threads of the JVM, dumped every 10 minutes and at shutdown |

A request seen on the console:

```text
WARN  [OpenAEV API] [http-nio-exec-3] [6a3c4dea…0b-7bd42e33…] ...debug.orm   : ORM GET /api/scenarios/sc-42: 13 queries (2 distinct), 7ms user=8f2b-user-helene
  N+1 SUSPECTED: 'select team_name from teams where team_id = ?' executed 12x (6ms total)
```

The same request in the SQL log:

```text
2026-04-12 23:36:42.179 INFO [trace=6a3c4dea... tenant=0e7c2f1a-tenant-grand-meridian user=8f2b-user-helene] sql success=true time=5ms statement=select team_name from teams where team_id = ? params=[{team_id=team-0}]
```

## Turn it on

:::steps
1. Set the debug mode in the environment of the `openaev` service. A platform runs in production when no `dev`, `test` or `ci` profile is active: there, the mode starts only with the production override too.

   ```yaml
   services:
     openaev:
       environment:
         OPENAEV_DEBUG_ENABLED: "true"
         OPENAEV_DEBUG_ALLOW-IN-PRODUCTION: "true"
         OPENAEV_DEBUG_AUTO-DISABLE-AFTER: 30m
         OPENAEV_DEBUG_OUTPUT-DIR: /var/debug
       volumes:
         - debug-data:/var/debug
   volumes:
     debug-data:
   ```

2. Recreate the container: `docker compose up -d openaev`.

   :::verify
   The log of the platform shows a banner saying that the debug mode is active, repeated every 5 minutes. Without the production override, it shows an error saying why the mode refused to start, and the platform runs as usual.
   :::

3. Reproduce the problem, and note the time and the page.
:::

`OPENAEV_DEBUG_AUTO-DISABLE-AFTER` stops the tracing by itself after 30 minutes, so it cannot be left on by mistake; until a restart, the platform keeps a small inactive hook on its database connections.

## Read the output

The SQL log is a stream to filter, not a file to read: it holds every statement of the platform, scheduled jobs included, which carry no trace (`[trace= tenant=]`).

:::steps
1. Find the request in the console log, with its time and its path, and copy its trace identifier.
2. Extract its statements, without folding the long lines:

   ```bash
   grep "trace=6a3c4dea" /var/debug/openaev-debug-sql.log | less -S
   ```

3. Look for the slowest `time=` and for the statements the ORM summary flagged as `N+1 SUSPECTED`.
4. Open the `.jfr` files with JDK Mission Control or `jfr print` to see where the CPU went.
:::

To keep only the statements of requests, without the scheduled jobs: `grep -E "trace=[0-9a-f]" openaev-debug-sql.log`. To check the masking: `grep MASKED openaev-debug-sql.log`.

## Turn it off

Remove `OPENAEV_DEBUG_ENABLED`, or set it to `false`, and recreate the container. The files stay in the output folder: send them to Filigran support if it asked for them, then delete them.

## Keep secrets out of the logs

Masking is on by default, `openaev.debug.masking.enabled`. A value is replaced by `***MASKED***` when its column or field is named like a secret, such as `password`, `token` or `client_secret`. It is also masked wherever it looks like a JSON Web Token, a `Bearer` or `Basic` authorization, a PEM private key or an email address. Both are best effort: a secret in a column with an unusual name and no known shape would still show.

For a guarantee, set `openaev.debug.masking.mask-all-parameters` (`OPENAEV_DEBUG_MASKING_MASK-ALL-PARAMETERS`) to `true`: every value is masked, only the column name and type stay, and the quoted strings of native queries are blanked too. Extend the lists with `openaev.debug.masking.sensitive-keys` and `openaev.debug.masking.value-patterns`.

## Every setting

All keys start with `openaev.debug.` and work only while `openaev.debug.enabled` is `true`. The [openaev.debug block](/reference/openaev/configuration/#openaev-debug) of the configuration reference gives their environment variables.

| Key | Default | What it sets |
| --- | --- | --- |
| `enabled` | `false` | Turns the debug mode on |
| `allow-in-production` | `false` | Lets the mode start in production |
| `auto-disable-after` | `0`, never | Stops the tracing after this duration, such as `30m` |
| `warning-interval` | `5m` | How often the log repeats that the mode is active |
| `output-dir` | `./logs/debug` | The folder of the files. A read-only container needs a mounted volume here |
| `sql.enabled` | `true` | Logs the SQL statements |
| `sql.slow-query-threshold` | `0ms`, all | Logs only the statements slower than this, such as `50ms` |
| `sql.max-parameter-length` | `200` | Cuts longer values, after masking |
| `sql.max-file-size` | `500MB` | Size at which the SQL log rolls over |
| `sql.max-history` | `7` | Days of SQL logs kept |
| `sql.total-size-cap` | `2GB` | Total size of SQL logs kept |
| `orm.summary-to-file` | `false` | Writes the ORM summary to `openaev-debug-orm.log` instead of the console, to keep it out of a centralized log pipeline |
| `jfr.enabled` | `true` | Records the JVM. Skipped when the Pyroscope agent is on (`pyroscope.agent.enabled`) |
| `jfr.settings` | `profile` | `profile`, detailed, or `default`, lighter |
| `jfr.duration` | `10m` | Interval between two dumps |
| `jfr.max-size` | `100MB` | Size cap of one recording |
| `jfr.max-age` | `1h` | Age of the oldest event kept |
| `jfr.max-dump-files` | `12` | Dumps kept |
| `jfr.max-total-dump-size` | `500MB` | Total size of the dumps kept |
| `masking.enabled` | `true` | Masks secrets and personal data |
| `masking.mask-all-parameters` | `false` | Masks every value |
| `masking.mask` | `***MASKED***` | The replacement text |

When the output folder is not writable, the platform keeps running. The JFR recording fails with `Debug mode: failed to start JFR recording`, and the SQL detail goes to the console with `Debug mode: log directory is not writable`.

:::example[Find why the scenario list of Grand Meridian is slow]
The security engineer of Grand Meridian Hotels reports that the list of scenarios of his tenant takes ten seconds to open since Example CERT imported the Velvet Harbor replays. The platform engineer of Example CERT turns the debug mode on for half an hour, on the production platform, keeping only slow statements and leaving the JVM alone:

```yaml
services:
  openaev:
    environment:
      OPENAEV_DEBUG_ENABLED: "true"
      OPENAEV_DEBUG_ALLOW-IN-PRODUCTION: "true"
      OPENAEV_DEBUG_AUTO-DISABLE-AFTER: 30m
      OPENAEV_DEBUG_OUTPUT-DIR: /var/debug
      OPENAEV_DEBUG_SQL_SLOW-QUERY-THRESHOLD: 50ms
      OPENAEV_DEBUG_JFR_ENABLED: "false"
    volumes:
      - debug-data:/var/debug
volumes:
  debug-data:
```

The engineer opens the list again. The console shows `N+1 SUSPECTED` on the request of the list, with one `SELECT` on the tags repeated for each scenario. The platform engineer sends the console lines and the statements of that trace to Filigran support, then removes the variables.
:::

## What's next

- [Troubleshoot OpenAEV](/openaev/administer/troubleshoot/).
- [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/), for the logs and their level.
- [Start the platform in safe mode](/openaev/administer/run-modes/), to rule out the scheduled jobs.
