# linear-skills

A Claude Code plugin marketplace by [Paweł Wlazło](https://github.com/pawelwlazlo) — skills for working with Linear and migrating issues into it.

## Installation

```
/plugin marketplace add pawelwlazlo/linear-skills
/plugin install linear-cli@linear-skills
/plugin install gitlab-to-linear@linear-skills
```

## Plugins

| Plugin | Description |
|--------|-------------|
| [linear-cli](plugins/linear-cli) | Complete command reference for the [`linear` CLI](https://linear.app/changelog/2024-10-29-linear-cli) v2.0.0. The agent loads this BEFORE running any `linear` command, so it doesn't have to call `--help` or guess parameters. |
| [gitlab-to-linear](plugins/gitlab-to-linear) | Imports open issues from a GitLab project into Linear via `glab` + `linear` CLIs. Preserves title, description, non-system comments, priority, assignee, and adds a backlink to the original GitLab issue. |

## Why a CLI-reference skill instead of an MCP?

Linear ships an MCP server, but in practice it bloats the agent's context with hundreds of tool schemas the user almost never needs. A skill that documents the `linear` CLI keeps the surface tiny: the description triggers the load, the body lists every flag, and the agent runs ordinary shell commands.

## Prerequisites

- **linear-cli** — install the [Linear CLI](https://linear.app/changelog/2024-10-29-linear-cli) and run `linear auth`.
- **gitlab-to-linear** — additionally needs the [`glab` CLI](https://gitlab.com/gitlab-org/cli) authenticated against the source GitLab host.

## Layout

```
linear-skills/
├── .claude-plugin/
│   └── marketplace.json        # marketplace catalog
├── plugins/
│   ├── linear-cli/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/linear-cli/SKILL.md
│   └── gitlab-to-linear/
│       ├── .claude-plugin/plugin.json
│       └── skills/gitlab-to-linear/SKILL.md
├── LICENSE
└── README.md
```

Each plugin can be installed independently.

## License

[MIT](LICENSE)
