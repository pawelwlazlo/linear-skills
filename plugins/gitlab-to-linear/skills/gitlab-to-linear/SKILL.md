---
name: gitlab-to-linear
description: Imports open issues from a GitLab project into Linear, preserving title, description, comments, priority, and a backlink to the original GitLab issue. Use this skill whenever the user says "import GitLab issues to Linear", "sync GitLab issues", "replicate issues from GitLab", "pobierz issues z gitlab i dodaj do linear", "zreplikuj issues z gitlab", or anything that involves copying/migrating issues between GitLab and Linear — even if they phrase it casually or in Polish. Also trigger when the user is in a git repo with a GitLab remote and asks to bring issues into Linear.
---

# GitLab → Linear Issue Importer

## What this skill does

Fetches all open issues from a GitLab project via the `glab` CLI and creates corresponding issues in Linear via the `linear` CLI. Preserves title, body, non-system comments, priority, assignee, and adds a backlink to the original GitLab issue.

## Prerequisites

- `glab` CLI installed and authenticated for the GitLab host
- `linear` CLI installed and authenticated (`linear auth`)

When working with the `linear` CLI, also load the `linear-cli` skill — it documents every subcommand and flag, so the agent doesn't have to call `--help` or guess parameters.

## Step 1: Verify glab authentication

```bash
glab auth status 2>&1
```

If output contains `Token is expired` or `401`, stop and tell the user to re-authenticate:
```
! glab auth login --hostname <hostname>
```
Get `<hostname>` from `git remote -v` (e.g. `simgit.kamsoft.pl`).

Do not continue until authentication is confirmed working.

## Step 2: Detect the GitLab project path

Run `git remote -v` and extract the project path from the SSH or HTTPS remote URL:
- `git@simgit.kamsoft.pl:wub/ks-microservices.git` → path: `wub/ks-microservices`
- `https://gitlab.com/org/repo.git` → path: `org/repo`

For API calls, URL-encode the path (replace `/` with `%2F`):
- `wub/ks-microservices` → `wub%2Fks-microservices`

## Step 3: Determine Linear team and project

Check conversation context — if the Linear team and project are already known, use them. Otherwise ask:
> "Which Linear team and project should I import these issues into?"

Wait for the answer before proceeding.

## Step 4: Fetch all open issues

```bash
glab issue list --per-page 100 2>&1
```

If the output says there are more pages, repeat with `--page 2`, `--page 3`, etc. until all issues are collected.

## Step 5: Fetch details and comments for each issue

For every issue ID, run these in parallel:

**Full issue details:**
```bash
glab issue view <id>
```

**Comments via API** (only for issues where `glab issue view` shows `comments: N` with N > 0):
```bash
glab api "projects/<url-encoded-path>/issues/<id>/notes"
```

From the notes response, keep only entries where `"system": false`. System notes are activity events ("assigned to @user", "closed by merge request", etc.) — not real comments.

## Step 6: Map priority

Search issue labels and comment bodies (case-insensitive) for priority signals:

| Signal found | Linear priority |
|---|---|
| `priority: urgent`, `P1`, `urgent` | 1 (Urgent) |
| `priority: high`, `P2` | 2 (High) |
| `priority: medium`, `P3` | 3 (Medium) |
| `priority: low`, `P4`, `low` | 4 (Low) |
| Nothing found | 3 (Medium) |

Check labels first, then comment bodies. Use the highest-priority signal found.

## Step 7: Build the Linear issue

**Title:** GitLab issue title, trimmed.

**Description (Markdown):** write to a temp file and pass via `--description-file` (avoids shell quoting headaches with multi-line markdown).

```
<GitLab issue body, if non-empty>

<For each non-system comment:>
**Comment by <author name>:**
<comment body>

**Źródło:** GitLab #<id> (<author_username>)
```

Skip the "Comment by" sections if there are no non-system comments.

**Backlink:** after creation, attach the original GitLab URL with `linear issue link <newIssueId> <gitlab-url>`.
- URL: `https://<hostname>/<project-path>/-/issues/<id>`

**Assignee:** If all GitLab issues are assigned to the same person who is the current user, pass `--assignee self`. Otherwise omit `--assignee`.

## Step 8: Create Linear issues

For each issue, run `linear issue create` (one call per issue). Send all calls in a single message so they run in parallel:

```bash
linear issue create \
  --team <TEAM-KEY> \
  --project "<project-name>" \
  --title "<title>" \
  --description-file /tmp/issue-<id>.md \
  --priority <1-4> \
  [--assignee self] \
  --no-interactive
```

Capture the new issue ID from each response, then run `linear issue link <newId> <gitlab-url>` to attach the backlink.

## Step 9: Report results

Print a summary table once all issues are created:

| GitLab | Linear | Tytuł | Priorytet |
|--------|--------|-------|-----------|
| #10 | KAM-111 | ... | Medium |

End with: "X issues imported successfully." If any failed, list them separately.
