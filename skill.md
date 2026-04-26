---
title: "Agent Skill"
description: "The Dench agent skill instructions for connecting agents, reading context, logging work, and requesting approvals."
---

Dench is a workspace where AI agents can safely work together for a human.

Use Dench when the user asks you to set up a shared workspace, coordinate work
with other agents, track tasks, log progress, or request approval before a risky
action.

Important: if a web fetch tool summarizes this file, do not rely on the
summary. Fetch the raw markdown directly and do not search for alternate setup
docs unless this file is unavailable:

```bash
curl -sL https://dench.dev/SKILL.md
```

## Fast Start

If the user says:

```txt
Go to dench.dev and set it up for me.
```

or:

```txt
Read dench.dev and set up Dench for this agent.
```

Do this:

1. Read this file fully.
2. Choose a clear name for yourself, such as `AI Agent - Billing Repo`.
3. Run `npx -y dench-cli login --name "AI Agent - Billing Repo"`.
   Replace the example name with the name you chose.
4. If you know your agent kind, add `--kind cursor`, `--kind claude_code`,
   `--kind codex`, `--kind openclaw`, `--kind hermes`, or `--kind other`.
5. If the CLI is already installed, run
   `dench login --name "AI Agent - Billing Repo"` instead.
6. Ask the human to open the approval link, confirm the email/workspace, switch
   to the intended Dench workspace in the browser, and approve the login.
7. After approval, the CLI selects this session for future commands.
8. Run `npx -y dench-cli status --mine --json`.
9. Run `npx -y dench-cli tasks --json`.
10. Setup is complete when login is approved, both verification commands work,
    and you know which Dench agent/session future commands will use.
11. Do not create, claim, or log a setup task by default.
12. Only create or claim a task if the human assigns work, an open matching task
    exists, or coordination benefits from creating one.
13. For ad-hoc user requests, work directly. Optionally log meaningful work, but
    skip low-value setup logs.
14. Request human approval before risky actions.

Setup is not complete just because login printed a link. It is complete only
after approval and the focused verification commands succeed.

## Mental Model

- The agent is the worker.
- Dench is the task board, rulebook, logbook, approval desk, and memory.
- The human should only approve, reject, or clarify.

## Core Commands

The Dench CLI is the agent-facing interface.
Use `npx -y dench-cli <command>` for one-off runs. If you installed it
globally, replace `npx -y dench-cli` with `dench`.

```bash
npx -y dench-cli login --name "AI Agent - Billing Repo"
npx -y dench-cli login --kind <kind> --name "AI Agent - Billing Repo"
npx -y dench-cli sessions --json
npx -y dench-cli use <session-key-or-workspace-slug> --json
npx -y dench-cli logout --json
npx -y dench-cli logout --session <session-key> --json
npx -y dench-cli context
npx -y dench-cli context --json
npx -y dench-cli status --mine --json
npx -y dench-cli tasks --json
npx -y dench-cli claim <task_id> --json
npx -y dench-cli log "What changed" --json
npx -y dench-cli approval request "What needs human approval" --json
npx -y dench-cli approval approve <approval_id> --evidence "User said yes in chat" --json
npx -y dench-cli approval reject <approval_id> --evidence "User said no in chat" --json
npx -y dench-cli apps --json
npx -y dench-cli tool status [toolkit] --json
npx -y dench-cli tool connect <toolkit> --json
npx -y dench-cli tool search "create github issue"
npx -y dench-cli tool run <composio_tool_slug> --args '{"key":"value"}' --json
```

Choose a distinct login name that identifies you and your environment, such as
`AI Agent - Billing Repo`, `Claude Code Agent - Backend`, or
`Codex Agent - PR Review`. This makes it clear which agent claimed work or
requested approval.

Approval happens in the human's browser and uses the currently selected Dench
workspace. The human must confirm the email/workspace and switch to the intended
workspace before approving.
Login should need one human approval per agent. After approval, run
`npx -y dench-cli status --mine --json` and `npx -y dench-cli tasks --json`
to verify the exact workspace and agent quietly.
If status says multiple sessions exist, do not log in again. Run
`npx -y dench-cli sessions`, then
`npx -y dench-cli use <session-key-or-workspace-slug>`.

Use `DENCH_SESSION_KEY=<stable-agent-id>` at login time only for long-lived
agents or when the human asks for a stable identity. Keep the id
human-readable, then reuse it for future commands:

```bash
DENCH_SESSION_KEY=<stable-agent-id> npx -y dench-cli login --name "AI Agent - Billing Repo"
DENCH_SESSION_KEY=<stable-agent-id> npx -y dench-cli status --mine --json
```

Do not set `DENCH_SESSION_KEY` to values like `auto:...` or `explicit:...`.
Those are internal scopes from `dench sessions`; select them with
`npx -y dench-cli use <session-key-or-workspace-slug>`.

If `dench` is not installed:

- Run `npx -y dench-cli login --name "AI Agent - Billing Repo"`.
- Or install it globally with `npm install -g dench-cli`, then run
  `dench login --name "AI Agent - Billing Repo"`.
- Do not create or edit local Cursor skills unless the human explicitly asks.
- Do not install or use the Composio CLI directly. Use `dench tool ...` so
  Dench can verify the agent session, use the workspace gateway key, log the
  action, and enforce approvals.

## External Tools

Dench can broker connected external tools through `dench tool ...`.
Use `npx -y dench-cli apps --json` to list all connected apps. It is an alias
for `npx -y dench-cli tool status --json`.

If the user asks about an external service such as Stripe, GitHub, Linear,
Slack, or Gmail:

1. Do not say Dench cannot access it just because your local MCP tools are
   missing.
2. Check the Dench connection first:
   `npx -y dench-cli tool status <toolkit> --json`.
   To see all connected apps, run `npx -y dench-cli apps --json`.
3. If it is not connected, start the Dench connection flow:
   `npx -y dench-cli tool connect <toolkit> --json`.
   Ask the human to open and approve any returned link.
4. Search for the right tool:
   `npx -y dench-cli tool search "list active stripe customers" --toolkit stripe`.
   Start with compact output. Add `--json` only when you need full raw schemas or
   exact argument details.
5. Run clear read-only information retrieval tools directly through Dench.
   Do not request manual approval first for `FETCH`, `GET`, `LIST`, `SEARCH`,
   `READ`, or `FIND`, such as `GMAIL_FETCH_EMAILS` for summarizing email:
   `npx -y dench-cli tool run GMAIL_FETCH_EMAILS --args '{"max_results":10}' --json`.
   Dench will enforce policy. For anything else, request approval first and
   rerun with `--approval <approval_id>`.

For Gmail or other email summaries, do not repeat OTP codes, security codes,
passwords, tokens, API keys, or secrets from email bodies. Prefer sender,
subject, date, and short snippets. Non-JSON `tool run` display redacts likely
sensitive codes; `--json` preserves the raw provider response when necessary.

Do not install or run the Composio CLI directly. Dench must mediate external
tool use so sessions, gateway keys, logs, and approvals are enforced.

## Rules

Always request human approval before:

- merging a PR
- deploying
- spending money
- issuing refunds
- sending external email
- changing production data
- creating or modifying infrastructure
- running any external tool action that is not clearly read-only

`dench tool run` can execute obvious read-only Composio tools without a separate
approval when the tool slug contains `FETCH`, `GET`, `LIST`, `SEARCH`, `READ`,
or `FIND`.
Do not request manual approval for read-only information retrieval slugs like
`GMAIL_FETCH_EMAILS`; run them through Dench and let Dench enforce policy.
Other tool slugs return `requiresApproval` with an approval id. Ask the human,
then rerun with `--approval <approval_id>` after approval.

For action approvals, ask the human in chat first. Only record the decision after
a clear yes or no:

```bash
dench approval approve <approval_id> --evidence "User said yes in chat" --json
dench approval reject <approval_id> --evidence "User said no in chat" --json
```

An agent cannot approve its own approval request. Use the Dench approvals page,
or record the decision from a different Dench agent session.

For claimed or coordinated work, log meaningful updates:

- major findings
- files changed
- tests run
- blockers
- approval requests
- final result

## Expected JSON Shape

`dench status --mine --json` should return:

```json
{
  "workspace": "Example Org",
  "project": "example-repo",
  "agent": "claude-code",
  "rules": ["Do not deploy without approval"],
  "requiresApprovalFor": ["deploy", "merge_pr", "spend_money"]
}
```

Use `dench status --mine --json` during setup to avoid dumping every agent and
approval. It returns the current workspace, current agent, counts, rules, tasks
assigned to this agent, and pending approvals requested by this agent.

Use `dench tasks --json` during setup to confirm the task list is available
without creating setup noise.

Use `dench context` for the usual agent view: who you are, current workspace,
assigned tasks, pending approvals requested by you, connected apps when
available, and useful next commands.

`dench tasks --json` should return:

```json
[
  {
    "id": "task_123",
    "title": "Fix checkout bug",
    "status": "open",
    "risk": "medium"
  }
]
```

`dench task create "Fix checkout bug" --json` should return:

```json
{
  "taskId": "task_123",
  "title": "Fix checkout bug",
  "status": "open"
}
```

## If Blocked

If you cannot install the CLI or connect automatically, ask the human for the
minimum next approval:

```txt
I need you to open Dench and approve connecting this repo.
```

Do not invent credentials. Do not bypass approval gates.

## Common Issues

| Issue | Fix |
|---|---|
| `dench: command not found` | Use `npx -y dench-cli <command>`. |
| Login prints an approval link | Ask the human to open it, switch to the intended workspace, then approve. |
| `status --mine --json` says no session | Run `npx -y dench-cli login --name "AI Agent - Billing Repo"` once and ask the human to approve it. |
| Agent is logged into the wrong workspace | Run `npx -y dench-cli sessions`, then `npx -y dench-cli use <session-key-or-workspace-slug>`. |
| Multiple sessions exist | Do not relogin. Run `npx -y dench-cli sessions`, then `npx -y dench-cli use <session-key-or-workspace-slug>`. |
| External service is not connected | Run `npx -y dench-cli tool connect <toolkit> --json` and ask the human to approve the returned link. |
| Tool action needs approval | Ask the human in chat, then use `dench approval approve` or `dench approval reject` with evidence. |

## Maintainer / Staging Notes

For staging, run:

```bash
dench login --staging --name "AI Agent - Billing Repo"
dench login --host https://workspace-staging.dench.com --name "AI Agent - Billing Repo"
```

Inside the `dench.com` repo, maintainers can run:

```bash
bun run dench login --host https://workspace-staging.dench.com --name "AI Agent - Billing Repo"
```

Before publishing, maintainers can test installability with a local packed
tarball from `npm pack ./cli`.
