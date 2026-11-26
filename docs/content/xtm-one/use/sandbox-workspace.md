---
title: Let an agent run code in a sandbox
description: Let an XTM One agent run Python with the code interpreter, keep its files in the sandbox workspace across calls and turns, work on a repository checkout and run its checks.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, developer, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-10
edition: ee
migrated_from:
  - xtm-one:/usage/sandbox-workspace/
---

The code interpreter runs Python for an AI agent: parse a log export, compute statistics on indicators, convert a file, test a detection rule. Every conversation, and every assignment run, has a **sandbox workspace**: a directory that keeps its files from one call to the next, across the turns of the conversation and across a turn that was interrupted and resumed. An agent can bring in a repository once, then read it, fix it and test it over as many calls as it needs, instead of redoing everything in one giant call. This page turns the code interpreter on, and explains what persists, how content gets in and out, how an agent works on a checkout, and the limits.

::diagram{id=xtm-one-sandbox-workspace-calls}

## Before you start

- On XTM One, the code interpreter is off by default. An administrator turns it on in :menu[Administration > Settings > Access and security], under **Security options**, by switching off **Disable the code interpreter**. The variable `DISABLE_CODE_INTERPRETER`, when set, wins and locks the switch: see the **Security switches** section of the [configuration reference](/reference/xtm-one/configuration/).
- The agent carries the **Code Interpreter** built-in tool, chosen in the **Built-in Tools** card of its configuration. With the General Assistant, switch it on for the conversation in **Session tools**.
- Code runs count toward your agentic quota.
- To run a repository's checks and start its services, see [Run the repository's checks](#run-the-repository-s-checks): the sandbox must run on OpenSandbox, with `CUSTOM_TOOL_SANDBOX=opensandbox` ([Run code in OpenSandbox](/deploy/configure/xtm-one/#run-code-in-opensandbox)).

## What persists, and what does not

| | Kept between calls | Note |
| --- | --- | --- |
| Files under `workspace/` | Yes | The directory `workspace/` of the code's working directory; its absolute path is in the `WORKSPACE_DIR` variable of the sandbox |
| Packages installed into `workspace/.venv` | Yes | Only when an administrator allowed them: see [Packages](#packages) |
| Variables and imports, by default | No | Every call runs in a fresh, isolated process |
| Variables and imports, with `session=true` | While the kernel lives | See [Keep variables between calls](#keep-variables-between-calls) |
| Files written elsewhere in the working directory | No | They become downloadable files of the conversation |
| `attachments/` | No | The conversation's files, copied in for the call |

::screenshot{id=xtm-one-sandbox-workspace-code-run alt="Night shift triage answering Paul Garnier after running code in its sandbox: 2 of the 4 events match the Sigma rule, on FD-PAR-01 and FD-PAR-03, and the rule velvet_harbor_loader.yml handed back from the workspace as a file card."}

Every call stays isolated: the sandbox has no network access and holds no credential, and the workspace only ever holds the files of its own conversation or run. Each result starts with a line saying how the call ran, whether the workspace was mounted, and which files were added, changed or removed; the agent also sees what the workspace holds before each call.

A background task the agent starts, or an agent it consults, begins with its own empty workspace: a `workspace/` path of the conversation means nothing there. To hand it a file, the agent turns it into a file of the conversation with `sandbox_get_file` and passes that file, attaches it to the work item, or puts the text in the task. A tool that reads a path only another of your workspaces holds refuses, and says which one holds it.

## Bring content in, and out

Reading a whole repository through the conversation costs every file twice: once when the tool returns it, once when the agent writes it into code. The agent can send content straight to the workspace instead:

| Tool | What it does |
| --- | --- |
| `to_workspace_path` | An option of the tools that fetch a file or text (reading a file, web fetch, the GitHub file tools, the download tools of integrations, the read operations of an integration connected from an OpenAPI document): the content is written to the workspace on the server, and the agent gets back its path, size and SHA-256 instead of the content |
| `sandbox_put_file` | Copies a file of the conversation (an upload, a download, a generated file) into the workspace |
| `materialize_repository` | Checks a repository out into the workspace through a connected integration (GitHub), with that integration's credential and repository scope: a branch, a tag or a commit, and optionally some paths (`sub_paths`). The files are written without a `.git` directory or any credential; see [Work on a checkout](#work-on-a-checkout) |
| `sandbox_get_file` | Turns a workspace file into a file of the conversation, which the agent hands to you or to another tool |
| `workspace_path` | An option of the GitHub commit tools: the file is sent straight from the workspace, without going through the conversation |
| `sandbox_list_files`, `sandbox_move_files`, `sandbox_remove_files` | List, move and rename, and clean the workspace |

A PDF or an Office document (PDF, DOCX, PPTX, XLSX, XLS) fetched with `to_workspace_path` also lands as its text, in a `.txt` file next to it; with a path that ends in `.txt`, only the text is written. A scan without a text layer lands as the file alone, and the answer says so.

When tool approvals are on for the agent, `materialize_repository`, `sandbox_checkout_diff` and `sandbox_checkout_rebase` ask for approval like the download tools they use; the other `sandbox_*` tools follow the code interpreter's approval.

## Read and change files without copying them into the conversation

The code interpreter can do anything with a file, but a script that prints a file back costs it in the conversation. The agent reads and changes the workspace with tools that answer only what it asked for:

| Tool | What it does |
| --- | --- |
| `sandbox_outline` | Lists the classes, functions and methods of a source file (the headings of a Markdown file), with their line numbers |
| `sandbox_read_file`, `sandbox_read_symbol` | Read a window of lines, or one definition whole by its name |
| `sandbox_grep` | Searches the workspace and answers the matching lines |
| `sandbox_edit_file` | Replaces exact text, or a range of lines, in one file; all the edits of a call apply or none does, and the answer is the diff |
| `sandbox_write_file` | Writes a new file; it refuses a path that exists, unless the agent asks to overwrite it |
| `sandbox_replace_in_files` | Replaces a regular expression across files, all or nothing, with a dry run to preview |
| `sandbox_apply_patch` | Applies a unified diff to a checkout |
| `sandbox_diff_files` | Compares two files, or merges three, with conflict markers where both sides changed the same lines |
| `sandbox_json_keys` | Sets, renames, removes or reads keys across translation catalogs (JSON), each file kept in its layout |
| `sandbox_playwright_trace` | Reads a Playwright `trace.zip`, such as a CI artifact: the failing action, its call log, console errors and requests |

## Work on a checkout

`materialize_repository` writes a record, `.xtm-checkout.json`, in the checkout's directory: the repository, the ref, the commit and the hash of every file. From it, the agent works on the checkout as on a clone, with no `.git` directory:

- `sandbox_checkout_status` lists the files changed, added or removed since the checkout, like `git status`. The agent reads it before a commit, and first after a resumed run.
- `sandbox_checkout_diff` shows those changes as a unified diff, like `git diff`.
- `sandbox_checkout_rebase` moves the checkout onto a new ref, for instance when `main` moved or a pull request conflicts, without losing its edits: lines changed on both sides are written between conflict markers, and the answer lists the files in conflict.
- `materialize_repository` with `add_paths` grows a narrow checkout with more paths of the same commit, and keeps the edits already made.
- `create_github_commit` with `from_checkout` commits exactly what changed in the checkout, the same set `sandbox_checkout_status` lists.

## Run the repository's checks

When XTM One runs its sandbox on OpenSandbox, `sandbox_run_command` runs a checkout's checks: its tests, linters and builds. It reads them from the repository's CI workflows (`.github/workflows`), its manifests and its `Makefile`; called without a check, it lists them. Each check runs in a container of its image, with no network and no credential. With `affected=true`, it runs only the tests the changes touch. A check takes up to 600 seconds by default, and at most an hour. A command still running when the call ends answers `running` with a `run_id`, which the agent checks again later, or stops.

A repository can declare its checks in `.xtm-one/checks.toml`, which then replaces what is read from its files. For example:

```toml
[check.backend-unit]
argv = ["python", "-m", "pytest", "-m", "not integration", "-q", "--junitxml=.xtm/pytest.xml"]
cwd = "backend"
timeout = 1800
image = "python"
report = "junit"
report_path = ".xtm/pytest.xml"
```

`image` names a key of `OPENSANDBOX_IMAGES`, such as `python` or `nodejs`; `report` is `junit`, `sarif` or `lines`, read from `report_path`, relative to `cwd`. An agent runs only the checks listed this way, declared or read from the repository: a command of its own is refused unless an administrator allowed it with `CODE_INTERPRETER_COMMAND_ALLOW_ARGV`.

The same file declares the services the repository can start, under `[service.<name>]`, such as a development server. `sandbox_start_service` starts one, waits until it is ready, and answers a `preview_url` the agent opens with the browser tool or `web_fetch`; `sandbox_service_status`, `sandbox_service_logs` and `sandbox_stop_service` follow it. A service stops after an hour by default, and at most 4 hours.

## Keep variables between calls

With `session=true`, the code runs in the conversation's kernel, a long-lived Python process where variables and imports persist from one such call to the next; `reset=true` starts a new kernel. The kernel ends, and its variables with it, after 15 minutes without use by default, when a cell runs past the call's timeout, or when the call reaches another server process. Files under `workspace/` stay in every case: keep in a file what a later step must not lose.

## Packages

When an administrator has set a package allow-list (`CODE_INTERPRETER_PACKAGE_ALLOWLIST`), the agent can ask for packages with `packages=["name==version"]`. The server downloads them from the configured index, checks each one against the index's SHA-256, and installs them into `workspace/.venv`, where they stay. The sandbox itself stays offline. Only listed packages and their listed dependencies are installed; the result names what was installed and what was not, and why.

## Limits and clean-up

- A workspace holds at most 2 GB and 20,000 files by default. A call that would go past either keeps nothing it wrote under `workspace/` that time, and says so: remove files and run it again.
- When a checkout would go past these limits, the clean checkouts of a deleted branch are removed first; with `prune_clean_checkouts`, every other clean checkout too. A checkout with a change is never removed.
- A conversation's workspace is deleted when the conversation is archived or deleted, and after 7 days without use.
- An assignment run's workspace is deleted as soon as the run ends (completed, failed, skipped or cancelled) or is deleted, unless the run worked on a work item still open: the next run of that work item, by the same agent for the same person, then takes over the workspace and goes on from its files. It is deleted once the work item is done, cancelled or deleted.
- A run that is not over loses its workspace after 24 hours without use, or after 7 days while it waits for a person.

An administrator changes these limits, the idle times, the kernel timeout and the allow-list in the **Code interpreter sandbox workspace** section of the [configuration reference](/reference/xtm-one/configuration/).

## When a turn is interrupted

A restart, an update or a recycled worker can stop the server while an agent works. The tool calls the turn completed are kept, and the workspace stays as the turn left it. Once XTM One is back, the turn resumes by itself: it receives those calls and their results, with a note naming the last completed call and what the workspace holds, and the agent goes on from there. It stops resuming by itself after 3 cuts in a row without progress, 12 hours after the first cut, or 100 cuts in all. A turn stopped for another reason, such as a crash, waits for you: **Resume**, or any message, goes on the same way. See [Get answers from an agent in the chat](/xtm-one/use/chat/#when-an-answer-takes-long-or-stops).

:::example[Test a Sigma rule against the night's logs]
Hélène Marchetti has switched off **Disable the code interpreter** for Grand Meridian Hotels. Sophie Laurent gives **Night shift triage** the **Code Interpreter** built-in tool and the skill **Write a Sigma rule**. On 12 April 2026, Paul Garnier attaches the EDR export `edr-PAR-FD-03-night.jsonl` (180 MB) and asks:

> Write a Sigma rule for the PowerShell loader Velvet Harbor used, then test it against this export and tell me how many events match.

The agent copies the export into the workspace with `sandbox_put_file`, writes the rule to `workspace/velvet_harbor_loader.yml`, and runs three calls: one to parse the export into `workspace/events.parquet`, one to apply the rule, one to count the matches per host. The second call fails on a field name; the third reuses the parsed file instead of parsing 180 MB again. The agent then hands the rule back with `sandbox_get_file`: `velvet_harbor_loader.yml` appears as a card under its answer, and in :menu[Work > Files].
:::

## What's next

- [Find the files agents created and received](/xtm-one/use/files/), where the files an agent hands back are listed.
- [Teach agents a procedure with skills](/xtm-one/build/skills/), to give an agent the procedure behind its code.
- [Connect agents to your tools with integrations](/xtm-one/build/integrations/), to connect the GitHub integration a checkout comes from.
- [Set what one agent run may spend](/xtm-one/use/run-budgets/), to bound a run that runs a lot of code.
