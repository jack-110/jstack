# jstack

Plugin for Cursor and Codex. Cursor reads `.cursor-plugin/`. Codex reads `.codex-plugin/`. Both hosts use the same `skills/` directory: `flow-chart/`, `technical-writing/`, and `socratic-tutor/`. Add other component files under the directories below when you are ready. Names are lowercase kebab-case.

## Layout

```text
jstack/
├── .cursor-plugin/
│   └── plugin.json          # Cursor manifest. `name` is required.
├── .codex-plugin/
│   └── plugin.json          # Codex manifest. Points at ./skills/.
├── skills/
│   ├── flow-chart/
│   │   └── SKILL.md         # Flow chart skill
│   ├── technical-writing/
│   │   ├── SKILL.md         # Technical writing skill
│   │   └── agents/openai.yaml
│   └── socratic-tutor/
│       └── SKILL.md         # Socratic tutor skill
└── README.md
```

With no `rules`, `skills`, `agents`, or `commands` paths in the manifest, Cursor discovers components from these default directories. A path you later set in the manifest replaces discovery for that component, so new files belong in the directory the manifest names.

## Manifest

`.cursor-plugin/plugin.json` identifies the plugin. `name` is the only required field. It is lowercase kebab-case, starts and ends with a letter or digit, and is the id used at install time.

| Field | Current value | Role |
| --- | --- | --- |
| `name` | `jstack` | Plugin id |
| `displayName` | `Jstack` | Label in Customize |
| `version` | `0.1.0` | Semantic version |
| `description` | Cursor plugin skeleton. | One-line summary |
| `author.name` | `jack-110` | Author |
| `repository` | this repo | Source URL |
| `keywords` | `cursor`, `plugin` | Discovery tags |

Optional later: `logo` (a relative path such as `assets/logo.svg`), `hooks` (`hooks/hooks.json`), `mcpServers` (`mcp.json` at the plugin root), and `variables` (names only; secret values are set in the dashboard under Plugins → Configure).

## Components

### Rules — `rules/*.mdc`

Always-on or file-scoped guidance. Frontmatter:

| Field | Type | Role |
| --- | --- | --- |
| `description` | string | Shown in the rule picker |
| `alwaysApply` | boolean | `true` injects the rule into every session. `false` keeps it available on request. |
| `globs` | string or array | File patterns. Used when `alwaysApply` is `false`. |

```markdown
---
description: Short description shown in the rule picker
alwaysApply: true
---

Rule body.
```

Rules do not have a `name` field. The filename is the rule id. Set `alwaysApply` to `false` and add `globs` when the rule should apply only to matching files.

### Skills — `skills/<name>/SKILL.md`

A workflow the agent can run. The directory name and the `name` field match. Frontmatter:

| Field | Role |
| --- | --- |
| `name` | Lowercase kebab-case, max 64 characters |
| `description` | What the skill does and when to use it. Written in third person. This is how the agent decides to apply the skill. |

```markdown
---
name: example-skill
description: What this skill does and when the agent should use it.
---

# Example skill

Steps go here.
```

Extra reference belongs in a sibling file such as `reference.md`, linked from `SKILL.md`, one level deep. Add `disable-model-invocation: true` when the skill should run only if named explicitly.

### Agents — `agents/*.md`

A subagent prompt. Frontmatter:

| Field | Role |
| --- | --- |
| `name` | Lowercase kebab-case id a parent agent passes when spawning it |
| `description` | When the parent should spawn this agent |

```markdown
---
name: example-agent
description: What this agent does and when a parent agent should spawn it.
---

# Example agent

Prompt goes here.
```

### Commands — `commands/*.md`

A slash command. The `name` field is the command, so `example-command` is invoked as `/example-command`. Frontmatter:

| Field | Role |
| --- | --- |
| `name` | Lowercase kebab-case command id |
| `description` | Shown when picking the command |

```markdown
---
name: example-command
description: What /example-command does.
---

# Example command

Steps go here.
```

## Add a component

1. Create the file at the path in the layout above.
2. For a skill, name the directory with the kebab-case id.
3. Set `name` (skills, agents, commands) to that same id.
4. Write `description` and the body.

## Load it locally

Cursor reads unpacked plugins from `~/.cursor/plugins/local`. A marketplace plugin with the same `name` takes precedence over this copy.

```bash
mkdir -p ~/.cursor/plugins/local/jstack
rsync -a --delete --exclude .git ./ ~/.cursor/plugins/local/jstack/
```

Reload the window, then open Customize. Components appear there after you add their files. On Enterprise, local imports also require Allow Local Plugin Imports under Dashboard → Settings → Security & Identity → Marketplace and Plugins.

Publish from a public git repository at [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish) when the plugin has real behavior. For several plugins in one repo, add `.cursor-plugin/marketplace.json` at the repository root and move this plugin into its own directory.

## Load it in Codex

`.codex-plugin/plugin.json` is the Codex manifest. It points `skills` at `./skills/`, so Codex loads the same three skills Cursor does. `technical-writing` stays explicit-only: Cursor reads `disable-model-invocation` in `SKILL.md`, and Codex reads `policy.allow_implicit_invocation: false` in `skills/technical-writing/agents/openai.yaml`. Invoke that skill as `$technical-writing`.

Install it into the user Codex home, not this repo. The copy is what Codex loads when you open another directory. From the repo root:

```bash
mkdir -p ~/.codex/local-marketplaces/personal/plugins
rsync -a --delete --exclude .git ./ ~/.codex/local-marketplaces/personal/plugins/jstack/
codex plugin marketplace add ~/.codex/local-marketplaces/personal
codex plugin add jstack@personal
```

The marketplace catalog for that install lives at `~/.codex/local-marketplaces/personal/.agents/plugins/marketplace.json`. After you change a skill here, run the `rsync` again and restart Codex. In any thread, ask for a sequence diagram or a tutoring session, or invoke a skill by name: `$flow-chart`, `$technical-writing`, `$socratic-tutor`.
