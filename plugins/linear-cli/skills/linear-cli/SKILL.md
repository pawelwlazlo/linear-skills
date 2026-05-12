---
name: linear-cli
description: Complete command reference for the `linear` CLI v2.0.0. Invoke this skill BEFORE running any `linear` command — it contains every subcommand, flag, and allowed value so you don't have to call `--help` or guess parameters. Use whenever the user asks to list, create, update, query, comment on, or otherwise interact with Linear issues, projects, teams, cycles, milestones, initiatives, labels, or documents. Also trigger when the user mentions Linear issue IDs (e.g. ENG-123), Linear projects, or wants to record decisions/update status in Linear.
---

# Linear CLI v2.0.0 — Command Reference

Global flag available on all commands: `--workspace <slug>` (uses stored credentials).

## issue (alias: i)

### issue id
Print the issue linked to the current git branch.

### issue mine / list / l
List your own issues.
```
linear issue mine [options]
  -s/--state <state>       triage|backlog|unstarted|started|completed|canceled  (default: unstarted)
  --all-states
  --sort <sort>            manual|priority
  --team <team>
  --project <name>
  --project-label <label>
  --cycle <name|number|active>
  --milestone <name>       requires --project
  -l/--label <label>       repeatable
  --limit <n>              default 50; 0 = unlimited
  --created-after <date>   ISO 8601 or YYYY-MM-DD
  --updated-after <date>
  -w/--web | -a/--app
  --no-pager
```

### issue query / q
Query across all issues with filters.
```
linear issue query [options]
  --search <term>
  --search-comments        requires --search
  --team <key>             repeatable; or --all-teams
  -s/--state <state>       repeatable: triage|backlog|unstarted|started|completed|canceled
  --all-states             (default)
  --assignee <username>    or -A/--all-assignees (default) or -U/--unassigned
  --sort <sort>            manual|priority  (priority not available with --search)
  --project <name>
  --project-label <label>
  --cycle <name|number|active>
  --milestone <name>       requires --project
  -l/--label <label>       repeatable
  --limit <n>              default 50; 0 = unlimited
  --created-after <date>
  --updated-after <date>
  --include-archived
  -j/--json
  --no-pager
```

### issue view / v [issueId]
View issue details (omit issueId to use current git branch issue).
```
  -w/--web | -a/--app
  --no-comments
  --show-resolved-threads
  -j/--json
  --no-pager
  --no-download            keep remote URLs instead of downloading attachments
```

### issue create
```
linear issue create [options]
  -t/--title <title>             required (or interactive)
  -d/--description <text>
  --description-file <path>      preferred for multi-line markdown
  -a/--assignee <assignee>       'self' or username/display name
  -p/--priority <1-4>            1=urgent 2=high 3=medium 4=low
  -s/--state <state>             by name or type
  -l/--label <label>             repeatable
  --team <team>
  --project <name|slug>
  --milestone <name>
  --cycle <name|number|active>
  --parent <TEAM-123>
  --due-date <YYYY-MM-DD>
  --estimate <points>
  --start                        set In Progress after creation
  --no-use-default-template
  --no-interactive
```

### issue update [issueId]
Same flags as `create` except `--start`, `--no-use-default-template`, `--no-interactive`.

### issue start [issueId]
Mark issue as In Progress.

### issue title [issueId]
Print title only.

### issue url [issueId]
Print URL.

### issue describe [issueId]
Print title + Linear-issue trailer (useful for commit messages).

### issue pr / pull-request [issueId]
Create GitHub PR prefilled with issue details.

### issue delete / d [issueId]

### issue comment
```
linear issue comment add [issueId]
  -b/--body <text>
  --body-file <path>        preferred for markdown
  -p/--parent <commentId>   reply to a comment
  -a/--attach <filepath>    repeatable

linear issue comment list [issueId]
linear issue comment update <commentId>
linear issue comment delete <commentId>
```

### issue attach <issueId> <filepath>
Attach a file to an issue.

### issue link <urlOrIssueId> [url]
Link a URL to an issue.

### issue relation
```
linear issue relation add    <issueId> <relationType> <relatedIssueId>
linear issue relation delete <issueId> <relationType> <relatedIssueId>
linear issue relation list   [issueId]
```

---

## project (alias: p)

```
linear project list
linear project view|v <projectId>   [-j/--json]
linear project delete <projectId>

linear project create
  -n/--name <name>          required
  -d/--description <text>
  -t/--team <key>           required; repeatable for multiple teams
  -l/--lead <lead>          username | email | @me
  -s/--status <status>      planned|started|paused|completed|canceled|backlog
  --start-date <YYYY-MM-DD>
  --target-date <YYYY-MM-DD>
  --initiative <id|slug|name>
  -i/--interactive
  -j/--json

linear project update <projectId>
  -n/--name | -d/--description | -s/--status | -l/--lead
  --start-date | --target-date | -t/--team
```

---

## project-update (alias: pu)

```
linear project-update create|c <projectId>
  --body <text>
  --body-file <path>
  --health <onTrack|atRisk|offTrack>
  -i/--interactive

linear project-update list|l <projectId>
```

---

## team (alias: t)

```
linear team list
linear team id
linear team create
linear team delete <teamKey>
linear team autolinks
linear team members [teamKey]
```

---

## cycle (alias: cy)

```
linear cycle list
linear cycle view|v <cycleRef>
```

---

## milestone (alias: m)

```
linear milestone list
linear milestone view|v <milestoneId>
linear milestone create
linear milestone update <id>
linear milestone delete <id>
```

---

## initiative (alias: init)

```
linear initiative list|ls
linear initiative view|v <initiativeId>
linear initiative create
linear initiative update <initiativeId>
linear initiative archive [initiativeId]
linear initiative unarchive <initiativeId>
linear initiative delete [initiativeId]
linear initiative add-project <initiative> <project>
linear initiative remove-project <initiative> <project>
```

---

## label (alias: l)

```
linear label list
linear label create
linear label delete <nameOrId>
```

---

## document (aliases: docs, doc)

```
linear document list|l
linear document view|v <id>
linear document create|c
linear document update|u <documentId>
linear document delete|d [documentId]
```

---

## auth

Manage stored credentials (interactive).

---

## api [query]

Raw GraphQL request against the Linear API. Prints schema with `linear schema`.

---

## Quick reference

| Concept | Values |
|---------|--------|
| State types | `triage` `backlog` `unstarted` `started` `completed` `canceled` |
| Priority | `1`=urgent `2`=high `3`=medium `4`=low |
| Project status | `planned` `started` `paused` `completed` `canceled` `backlog` |
| Project health | `onTrack` `atRisk` `offTrack` |
| Cycle reference | name, number, or `active` |
| Assignee | username, display name, or `self` |
| Lead | username, email, or `@me` |
| File flag pattern | `--description-file` / `--body-file` → preferred for multi-line markdown |
| Issue ID format | `TEAM-123` (team key + dash + number) |

## Tips

- Omitting `[issueId]` on most `issue` subcommands falls back to the issue from the current git branch.
- `--description-file` / `--body-file` avoid shell quoting headaches with markdown — write content to a temp file and pass the path.
- `-j/--json` on `issue query` / `issue view` / `project view` gives machine-readable output for scripting.
- `--no-interactive` on `issue create` skips all prompts — useful in scripts.
- `LINEAR_DEBUG=1` shows full error stack traces.
- `LINEAR_ISSUE_SORT` env var sets default sort for `issue mine`.
