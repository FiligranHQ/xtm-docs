---
title: Give agents built-in and custom tools
description: Know the built-in tools an agent can use, then write a JavaScript or Python tool of your own, declare its tests, sync tools from a GitHub or GitLab repository, and give them to an agent.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, developer, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-19
edition: ee
migrated_from:
  - xtm-one:/usage/resources/tools/
---

A tool is an action an agent can take beyond writing text: fetch a web page, run Python, generate a file, open a signed-in browser. XTM One ships built-in tools, and lets you write custom tools: a short JavaScript or Python function with declared inputs, which an agent calls like any other tool and which runs in a sandbox. This page shows what the built-in tools give an agent, then writes, tests and attaches a custom tool, and syncs custom tools from a Git repository. Integrations and MCP servers bring the tools of whole external services: see [Connect agents to your tools with integrations](/xtm-one/build/integrations/) and [Connect an MCP server](/xtm-one/build/mcp-servers/).

::diagram{id=xtm-one-build-tools-call}

## The built-in tools

:menu[Tools & Integrations > Tools] opens on **Built-in Tools**: one card per tool the platform offers, with its category and the number of functions it grants on this platform. A built-in tool is a set of related functions: **Web**, for example, reads pages, saves images and drives a browser. Opening a card shows the **Agents Using This Tool** and its runs. Search the list by a tool's name or description; the view switch shows it as cards or as a table, with its **Category** column. You turn a built-in tool on for an agent in its **Configuration**, **Tooling** section, **Built-in Tools** card, or in the **Tools** step of the creation wizard.

| Tool | What it gives the agent |
| --- | --- |
| **Platform** | Manage the platform from a conversation: create and configure agents, assignments, knowledge bases, skills, variables, integrations, MCP servers, custom tools and fleets. It acts as the person the agent works for, and offers only what that person may do. |
| **Organization Profile Editing** | Update the [organization profile](/xtm-one/administer/organization-profile/) of :menu[Administration > Settings > Organization], such as its crown jewels, tech stack or risk appetite, for the administrator the agent acts for. Each change is a new version authored by that administrator, which the history can roll back. It can also propose a change for an administrator to review. Only an administrator adds it to an agent: anyone else sees it locked. Its functions are offered only when the agent acts for an administrator, and every agent reads the profile without it. |
| **Web** | Read public web pages and documents, such as a link a user shares, save a public image to the conversation, and drive a headless browser to navigate, click, fill forms and take screenshots; with [browser profiles](/xtm-one/build/browser-profiles/), signed in to your applications. |
| **Files** | Generate downloadable files, read and edit the conversation's files, render HTML to an image, and generate images from a text prompt. The image model is a platform-wide setting an administrator changes in **Image Generation Model**, on the tool's **Configuration** tab. |
| **Report Generation**, **Presentation Generation** | Executive reports and slide decks, branded by [templates](/xtm-one/build/templates/). |
| **Design** | Find a [template](/xtm-one/build/templates/) and read its layouts and assets, write one from a conversation, and read the exact palette and typefaces of a PDF, a PowerPoint file or an HTML page. |
| **Code Interpreter** | Run Python in a sandbox for calculations and charts. Off by default on XTM One: an administrator turns off **Disable the code interpreter** in :menu[Administration > Settings > Access and security > Security options]. |
| **Threat Intelligence** | Check the structure and references of a STIX 2.1 bundle before it is imported, compare two bundles, analyse a timeline, and run statistical tests and FIMI metrics, each result with its formula. |
| **STIX Extraction** | Turn threat reports, attached or pasted, into one STIX 2.1 bundle in OpenCTI's ontology. Each entity, relationship, indicator and ATT&CK technique is tied to the sentence that states it. Off by default, and not part of **Threat Intelligence**: see below. |
| **Coordination** | Coordinate agents across runs: a shared work queue, a key-value store, a large working corpus. |
| **Traces** | Read the agent's own traces, to review how its runs went. It also reads other agents' traces: for an administrator, those of any agent they can see; for anyone else, those of the agents listed in its **Trace supervision allow-list**, which shows under the card while you edit it. |
| **User Administration** | Administer accounts and groups as :menu[Administration > Users and groups] does: list and read accounts, disable them and enable them again, add and remove group members, and work the deactivation review. Only an administrator adds it to an agent, and its functions are offered only when the agent acts for an administrator. |
| **Platform Insights** | Answer questions about the platform's AI usage and cost, its objects and its audit trail, with charts: see [Get answers about the platform's own activity](/xtm-one/use/platform-insights/). |
| **Computer Use (Runners)** | Drive a computer enrolled with XTM One Desktop: see [Let agents act on a computer with XTM One Desktop](/xtm-one/build/runners/). |

Give an agent only the tools its job needs: every tool is something it can act through. When the wizard's **Tools** step is left empty, the agent starts with the platform's default tool set, which you can change on its page afterwards.

**STIX Extraction** reads each report passage by passage with the agent's own model, so one report costs many model calls and a long one takes minutes. These calls count in the run's budget: when it runs out, the bundle comes back marked incomplete. One call stops after 30 minutes, or sooner when the run has less time left, and names the reports it could not finish. The tool never runs on the General Assistant: switch it on only for an agent that extracts. The built-in agent **CTI STIX Extractor (Grounded)** carries it alone and answers with the bundle the tool returns. That agent takes no chat and is bound to no intent, so OpenCTI's imports still go to the harvester agents. See [Choose which agent answers each AI feature](/xtm-one/administer/intents/).

**User Administration** disables an account through the same steps as **Disable** on the person's page: see [What disabling an account does](/xtm-one/concepts/access/#what-disabling-an-account-does). It previews what a deactivation would do, and acts only when the agent explicitly asks it to. It refuses your own account, the bootstrap account, the last active administrator, and an account protected as a break-glass or service account. No screen sets that protection. A group your identity provider manages through SCIM stays read-only. Each change is recorded in **Audit logs** as the administrator's, with the agent and the run.

When an MCP server bound to the agent offers a tool with the same name as a built-in, integration or custom tool, the agent keeps the platform's tool and the server's is left out. A server that is another XTM One, for example, offers its own agent and skill tools: they never take the place of this platform's.

### Keep only some functions of a tool

An agent can carry a built-in tool without all of its functions: an agent that only reads web pages has no use for the browser.

:::steps
1. Open the agent, its **Configuration** tab, then **Tooling**.
2. In the **Built-in Tools** card, click **Edit**. Each enabled tool with more than one function shows **All functions** under its name.
3. Click it. The **Function Scope** dialog lists the tool's functions: tick only the ones the agent needs (the search, **All functions** and **None** help), then click **Apply**.
4. Click **Save**. The button now reads how many functions the agent keeps, for example "2 of 3 functions selected", and outside edit mode the tool shows the same count next to its name.
:::

::screenshot{id=xtm-one-build-tools-function-scope alt="The Function Scope dialog of Traces for the Crisis briefer agent: Inspect my traces and Get trace payload checked, Inspect agent traces unchecked, 2 of 3 functions selected, with the search, the All functions and None buttons, and Apply."}

An agent set up with one of the earlier, narrower tools, such as **Browser** or **Inspect My Traces**, keeps exactly what it had: it shows the tool that includes it now (**Web**, **Traces**), limited to those functions. Open the scope to give it the rest.

An assignment can narrow the functions further for its own runs, never beyond the agent's: on its **Prompt & Bindings** tab, the **Built-in Tool Scope** card lists the agent's tools that keep more than one function. Click **Edit**, then a tool's button, choose in the same dialog, and click **Save**.

A function that a release adds to a tool is not given to an agent that a person narrowed to some of its functions: nothing is granted on its own. After the upgrade, the agent's owner gets one notification, under **Built-in tools**, naming the new functions and the agents left without them. It opens the first agent's **Tooling** section, where a notice reads, for example, "1 new built-in tool is outside this agent's allowlist" and lists each function by its identifier:

- **Add** gives the function to the agent, and **Dismiss** leaves it out. With several functions, **Add all** and **Dismiss all** act on every one. Both need AI switched on.
- A dismissed function is never listed again for that agent. The tool's **Function Scope** dialog still offers it.
- The notification is marked read once nothing it names is left to add or dismiss. For the other agents it names, open their **Function Scope** dialog.

A list of functions that XTM One wrote itself, and no person chose, is not held back. That is the case when a fleet role gives a member some functions of a tool it lacked. It is also the case when saving an agent turns an earlier, narrower tool into the tool that includes it. A release adds its new functions to such a list on its own, with no notification. The list becomes a person's choice once someone saves a different one for that tool.

## Before you write a custom tool

- Creating a custom tool requires the Enterprise Edition. Custom tools are not an AI feature: authoring them keeps working with AI switched off.
- Custom tools are off by default on XTM One. An administrator turns off **Disable custom tools** in :menu[Administration > Settings > Access and security > Security options], or sets `DISABLE_CUSTOM_TOOLS` to `false`. While they are off, the **Custom Tools** and **Repositories** tabs and **Add Tool** are hidden, and an agent never calls one.
- Tools run in a hardened Docker container when the platform can start one, otherwise in a restricted subprocess. A platform engineer chooses with `CUSTOM_TOOL_SANDBOX` (`auto` by default). See [Security switches](/reference/xtm-one/configuration/#security-switches) in the configuration reference.

## Write and test a custom tool

:::steps
1. Open :menu[Tools & Integrations > Tools], select **Custom Tools**, and click **Add Tool**. The **Create Custom Tool** page opens.
2. Fill **Tool Name** (required), the name the model calls the tool by (for example `booking_lookup`). Optionally add **Icon URL (optional)** and **Tags**.
3. Write **Description** for the model: what the tool does and when to call it. The model decides to call a tool from this text alone.
4. Under **Input Parameters**, click **Add Parameter** for each input: **Name**, **Type** (`string`, `number`, `boolean` or `integer`), **Description** and **Required**. The model fills them when it calls the tool.
5. Choose the **Language** and write the function:
   - **JavaScript Function**: read a parameter as `$param_name`, call web services with `fetch`, and write the result to the standard output (`console.log`).
   - **Python Function**: read a parameter as a plain variable (`param_name`) and `print()` the result. Network libraries are blocked: keep Python tools for processing and formatting data.

   To reuse a stored value such as an API key, click **Variable** and pick it rather than typing it in the code: see [Store values agents reuse in variables](/xtm-one/build/variables/). **Generate with AI** writes the code from the name, the description and the parameters you declared; on existing code it reads **Improve with AI**. Nothing is saved until you save the tool.
6. Set **Execution Timeout (seconds)**, from 1 to 3600: the platform stops a run still going after that time. The default is 300 seconds.
7. As an administrator, turn on **Company-managed tool** to offer it to every user (only administrators edit it then).
8. Click **Create Tool**. The tool's page opens.
9. Click **Test Tool**, write the **Arguments (JSON)** and click **Run**. The dialog shows **Success** or **Failed** with the output, exactly what an agent would read. Turn on **Dry run** first to run it with `dry_run` set to true: see [Declare the tests a tool must pass](#declare-the-tests-a-tool-must-pass).
:::

::screenshot{id=xtm-one-build-tools-create alt="The booking_lookup tool in edit mode, with the same fields as the creation page: its description for the model, the required reference parameter, the JavaScript function that reads $reference and sends the BOOKING_API_TOKEN variable by its reference, and a 30-second timeout."}

The tool's page has four tabs:

- **Overview**: its description, language, timeout, parameters and **Agents Using**.
- **Configuration**: everything above, plus the sharing and **Hide from chat**, which keeps the tool out of the chat's tool picker while agents can still use it.
- **Tests**: the tests the tool declares, and their last run (see below).
- **Activity**: the runs of the agents that use it, in a table (**Run**, **Status**, **Type**, **Agent**, **Started**, **Duration**). Filter them by status on the toolbar or search them, and click a row to open the run.

### Declare the tests a tool must pass

A tool can declare its own tests, which run it in its sandbox and check what it prints: data cases, written in JSON, and test code in the tool's language. Run them after each change, before an agent calls the new version. A tool synced from a repository declares them in the repository, and a new version is activated only when they pass.

:::steps
1. On the tool's page, open the **Tests** tab.
2. In the **Declared tests** card, click **Edit**.
3. In **Data cases (JSON)**, write a list of cases. Each case runs the tool once with its inputs and checks the output (keys below).
4. Optionally, in **Test code (JavaScript)** or **Test code (Python)**, write test functions that call the tool themselves.
5. Click **Save**.
6. In the **Tests** card, click **Run all tests**. A case's own button runs that case alone, and **Dry run only** runs only the dry-run cases.
:::

::screenshot{id=xtm-one-build-tools-tests alt="The Tests tab of the booking_lookup tool before any run: the Tests card reads Not run yet, 2 data cases and no test code, with Dry run only, Run all tests and one button per case, known booking and unknown booking, above the Declared tests card and its dry-run note."}

The card shows the outcome, **Tests passed**, **Tests failed** or **Tests errored**, then how many tests passed, failed, errored or were skipped, and one row per case or test with its time. A failed row shows why it failed. **Run tests**, at the top of the tool's page, runs them all too and says **Tests passed** or **Some tests did not pass**.

| Key of a data case | What it does |
| --- | --- |
| `name` | Required, unique among the tool's cases, up to 200 characters. It names the case's button. |
| `inputs` | The parameters of the run, as an agent sends them. |
| `dry_run` | `true` runs the case with `dry_run` set to true. `false` by default. |
| `expect` | What the output must hold, every check given: `equals` (the whole output), `contains` and `not_contains` (a text or a list of texts), `regex` (a regular expression), `json_subset` (values the JSON output must contain). |
| `expect_error` | `true`: the case passes only when the tool fails. `error_contains` also checks the error message, and sets `expect_error`. |

A tool declares at most 50 data cases and 200,000 characters of test code. Test code has the standard library only:

- In Python, write `test_*` functions or `unittest` classes that call `run_tool(inputs, dry_run=True)`.
- In JavaScript, write `test(name, fn)` or `it(name, fn)` with `assert` (from `node:assert`), and call `await runTool(inputs, { dryRun: true })`.

Both return what the tool printed, and raise or throw `ToolError` when the tool fails.

Every run of a tool receives `dry_run` (`$dry_run` in JavaScript): false when an agent calls it, true for a dry-run case or a **Test Tool** run with **Dry run** on. A tool that writes, for example one that disables an account or opens a ticket, must check it and skip its changes: the sandbox cannot do it for the tool. **Dry run only** skips the cases that are not dry runs, and sets `dry_run` on every call of the test code.

The **Custom Tools** tab lists the tools you can see, as cards or as a table with their **Language**, **Parameters**, **Timeout**, **Calls**, **Tags** and **Updated** date. Search them, narrow them with **Mine**, **Company**, **Shared** or **All**, and filter them by tag; **Clear all filters** resets the search and the tags. **Columns** hides the columns you do not need, and your browser keeps that choice and the page size for your next visit.

## Give it to an agent

On the agent's page, open **Configuration**, then the **Tooling** section. In the **Custom Tools** card, click **Edit**, select the tool and **Save**. The agent can call it from its next message or run. To change who may reuse the tool, see [Share agents and resources](/xtm-one/build/sharing/).

:::example[Look up a guest booking for the night shift]
Paul Garnier, SOC analyst on the night shift at Grand Meridian Hotels, often needs to know whether a booking named in a phishing email exists. Sophie Laurent, the AI engineer, writes a tool for the **Night shift triage** agent against the booking platform's internal API:

| Field | Value |
| --- | --- |
| **Tool Name** | `booking_lookup` |
| **Description** | `Checks whether a booking reference exists in the Grand Meridian booking platform and returns the hotel, the arrival date and the booking channel. Call it when an email or an alert names a booking reference.` |
| **Input Parameters** | `reference`, `string`, `The booking reference, for example GM-2026-118734`, **Required** on |
| **Language** | JavaScript |
| **Execution Timeout (seconds)** | `30` |

```javascript
const response = await fetch(`https://booking.grand-meridian.example/api/bookings/${encodeURIComponent($reference)}`, {
  headers: { Authorization: `Bearer {{var_id.7f3c2a10-5b8e-4d21-9a64-0c1e8b2f4d75}}` }
});
if (response.status === 404) {
  console.log(JSON.stringify({ found: false }));
} else {
  const booking = await response.json();
  console.log(JSON.stringify({ found: true, hotel: booking.hotel, arrival: booking.arrival, channel: booking.channel }));
}
```

The reference in the header is the secret variable `BOOKING_API_TOKEN`, inserted with **Variable**: the token never appears in the code, and traces show `[secret]`. She tests it with `{"reference": "GM-2026-118734"}`. On the **Tests** tab, she declares two data cases, a booking that exists and one that does not, so that every later change is checked. The tool only reads, so neither case needs a dry run:

```json
[
  {"name": "known booking", "inputs": {"reference": "GM-2026-118734"}, "expect": {"json_subset": {"found": true}}},
  {"name": "unknown booking", "inputs": {"reference": "GM-2026-000000"}, "expect": {"json_subset": {"found": false}}}
]
```

**Run all tests** reads **Tests passed**, and she adds the tool to **Night shift triage**. When an alert quotes a fake summit booking like the one Velvet Harbor sent on 9 January 2026, the agent answers that no such booking exists.
:::

:::xtm-one[Packaged skills bring their own tools]
A skill package can carry scripts: importing it creates one custom tool per script, which follows the skill onto every agent it is attached to. See [Teach agents a procedure with skills](/xtm-one/build/skills/).
:::

## Sync tools from a GitHub or GitLab repository

Keep your custom tools in a Git repository, reviewed like the rest of your code, and let XTM One sync them. Each directory of the repository that holds a `tool.yaml` becomes a custom tool. A new or changed tool is activated only when its declared tests pass. Synced tools are read-only in XTM One: you change them in the repository.

A public repository needs nothing more. A private one, or a project on a self-managed GitLab, is read through a connected GitHub or GitLab integration: see [Connect agents to your tools with integrations](/xtm-one/build/integrations/). The sync runs unattended, so the integration must use one shared account: one whose members each connect their own account is refused.

:::steps
1. Open :menu[Tools & Integrations > Tools], select **Repositories**, and click **Add a repository**.
2. Fill **Name**, choose the **Host**, **GitHub** or **GitLab**, and type the **Repository**: `owner/repository` on GitHub, `group/project` on GitLab, or its URL.
3. Optionally, set the **Branch, tag or commit** to read (the default branch otherwise), and a **Sub-directory** (the whole repository otherwise).
4. Choose the **Credential**: **None: a public repository**, or a connected integration of the same host.
5. Choose the **Automatic sync**: **Manual only**, **Every 6 hours**, **Every 12 hours**, **Daily** (the default) or **Weekly**.
6. As an administrator, turn on **Company-managed** to make its tools company-managed: visible to everyone, edited by administrators only. It cannot be changed later.
7. Click **Add and sync**. The first sync starts at once.
:::

A repository you add is yours: you alone see and manage it, and its tools are yours. Administrators see every repository, and only they manage a company-managed one. **Edit** changes everything but the host and **Company-managed**.

**Repository layout**, in the dialog, shows an example. In each tool's directory, next to `tool.yaml` (or `tool.yml`), sits the code, `main.py`, `main.js` or `index.js`, and optionally the test code, `test_main.py`, `main.test.js` or `test.js`. A file larger than 512 KB cannot be read, and a repository that declares more than 200 tools fails to sync.

| Key of `tool.yaml` | What it sets |
| --- | --- |
| `name` | Required: the tool's name, as the model calls it. |
| `description` | What the tool does and when to call it, for the model. |
| `input_schema` | The parameters, as a JSON Schema of type `object`. |
| `timeout` | From 1 to 3600 seconds, 300 by default. |
| `language` | `python` or `javascript`. Without it, the code file's extension decides. |
| `entry`, `test_file` | The code and the test code, when their names are not the default ones. |
| `tests` | The data cases, as in [Declare the tests a tool must pass](#declare-the-tests-a-tool-must-pass). |
| `tags`, `icon_url`, `hide_from_chat` | As on the tool's page. |

Each sync reads the latest commit of the branch, tag or commit you set:

- An unchanged tool is left as it is.
- A new or changed tool runs its tests first. It is activated when they pass, or when it declares none, and then reads **No tests**.
- A version whose tests fail is rejected: the active version stays, and a new tool is not created.
- A tool gone from the repository is disabled, never deleted. It is enabled again when it comes back.

Each repository's card shows its status, **Never synced**, **Synced**, **Partly synced** (a tool was rejected or could not be read) or **Sync failed** with the error. It shows the reference, the sub-directory, the interval, the number of tools and the last commit read. **Sync now** syncs it at once. **Last sync report** counts what the sync did, for example "1 created · 2 updated · 4 unchanged · 1 rejected by their tests · 0 disabled", and lists the tools of each kind, with the tests a rejected one failed.

A synced tool's page reads **Synced from** and the repository's name, which links to the repository. Its name, code, parameters and tests change in the repository: its **Configuration** keeps only the sharing and **Hide from chat**. To edit the tool in XTM One, click **Detach** and confirm: it becomes an ordinary tool, which the repository no longer updates. A synced tool cannot be deleted before it is detached. **Sync it again**, next to a detached directory in the report, lets the next sync read that directory again.

To delete a repository, click its delete button and confirm. Its tools stay, detached and editable, unless you turn on **Also delete its tools**: then agents lose them.

## Check that it works

The tool's **Overview** counts the agent under **Agents Using**, and its **Tests** tab reads **Tests passed** after the last run. In a chat with the agent, ask the question the tool answers: the answer shows the tool call and its result. A tool that times out or fails returns its error to the agent, which says so.

## What's next

- [Connect agents to your tools with integrations](/xtm-one/build/integrations/), for services XTM One already knows.
- [Connect an MCP server](/xtm-one/build/mcp-servers/), for a whole set of tools served by another system.
- [Create an agent](/xtm-one/build/agents/).
