# Auditing Agent Config

A portable agent skill that audits your Claude setup for drift — redundant skills, forgotten plugins, bloated CLAUDE.md files, session-start pile-up, and MCP servers nobody uses. It reports; it doesn't touch anything unless you tell it to.

It is plain Markdown, so it runs in any harness that supports skill-style instructions.

## Installation

### Skills CLI

```
npx skills add keshavjoglekar/auditing-agent-config
```

Install globally, across every project:

```
npx skills add keshavjoglekar/auditing-agent-config -g
```

### Claude Code plugin

```
/plugin marketplace add keshavjoglekar/auditing-agent-config
/plugin install auditing-agent-config@auditing-agent-config
```

### Claude desktop app / claude.ai

Download [`auditing-agent-config.zip`](https://github.com/keshavjoglekar/auditing-agent-config/releases/latest/download/auditing-agent-config.zip), then Settings → Capabilities → Skills → upload.

### Manual

```bash
mkdir -p ~/.claude/skills/auditing-agent-config
curl -fsSL https://raw.githubusercontent.com/keshavjoglekar/auditing-agent-config/main/SKILL.md \
  -o ~/.claude/skills/auditing-agent-config/SKILL.md
```

Restart your agent afterward. Skills load at session start.

## Usage

```
Audit my Claude setup. Report only, don't change anything.
```

It gathers what it can reach, asks you to paste output from the interactive commands it can't run itself, and returns a report: findings ranked by impact, then suggested improvements, then what it checked and found fine.

Re-run it every few months, before installing something that might overlap what you have, or whenever a skill stops triggering when you expect it to.

## Why config drift happens

Skill libraries accumulate cruft exactly like codebases do. Nobody removes the old thing when they add the thing that replaced it. Six months in, you have four tools that all claim to design UIs, a plugin firing a large instruction block at the start of every session, and no idea which of those is costing you anything.

Most guidance covers configuring well *from scratch*. This is the audit pass for the setup you already have.

## What it catches

| # | Failure mode | What it looks like | What the audit tells you |
|---|---|---|---|
| 1 | **Duplicate / shadowing skills** | Same skill visible twice under different namespaces | Which copy is real, which inventory it lives in, how to remove it |
| 2 | **Session-start pile-up** | 3–4 systems all injecting context before you type | Every actor claiming turn one, and which you actually need |
| 3 | **Overlapping coverage** | 14 skills competing to handle "design this UI" | Which ones have never fired, ranked by usage count |
| 4 | **Dead plugins** | `usageCount: 0` across 55 startups | Named, with the number, so the cut is defensible |
| 5 | **CLAUDE.md bloat** | 279 lines loading in full, every session | Line counts, what to move to `rules/` or a skill |
| 6 | **Volatile facts in CLAUDE.md** | "Currently job hunting" written into always-on context | Stable facts stay; changeable facts belong in memory |
| 7 | **Scope misplacement** | Client-specific rules in the global file | What to move, which direction, why |
| 8 | **Over-broad descriptions** | `"plan, build, create, fix, review, improve..."` | Why that skill fires on every request and collides |
| 9 | **MCP servers at odd scope** | `shopify-dev` loading in every project | Plus: disabling an MCP server does *not* reclaim its context |
| 10 | **Orphaned telemetry** | Usage entries for skills you uninstalled months ago | Named as history, never as something to "remove" |
| 11 | **Missing config** | No global CLAUDE.md despite five projects sharing conventions | What to add, grounded in evidence from your own setup |

Every removal it suggests names **which inventory the item lives in** and **how to remove it**. Every improvement it suggests cites **the evidence in your config that motivated it**. No evidence, no suggestion.

## The part that took longest to get right

There is no single list of "your skills." There are several inventories, and they disagree:

| Inventory | Where it actually lives | How you remove something |
|---|---|---|
| Account skills | Claude.ai account | Skills panel in the app |
| Plugins, and the skills they inject | Installed from a marketplace | `claude plugin disable <name>@<mkt>` |
| Local user skills | `~/.claude/skills/<name>/` | Delete the folder |
| Local project skills | `<repo>/.claude/skills/<name>/` | Delete the folder |
| Account connectors | Settings → Connectors — **in no file on disk** | Settings panel |

A running session shows you all of these mixed together. Plugin-injected skills appear under a namespace that makes them look like local files you could `rm` — you can't; they come back until the plugin is disabled. Usage telemetry persists after uninstall, so it lists things that no longer exist.

An early version of this skill confidently recommended deleting a pile of skills that weren't where it thought they were. The current version reconciles every inventory before recommending anything, and never suggests removing something it hasn't located first. That reconciliation step is most of what this skill is.

## What it does not do

- **It can't see your account connectors.** Those live in Settings, not on disk. It names them as a blind spot and points you at the right panel rather than guessing.
- **It can't run interactive commands** like `/context` or `/mcp`. It asks you to paste the output.
- **It doesn't judge whether a skill is *good*** — only whether it's redundant, misplaced, unused, or badly scoped.
- **It won't change anything on its own.** Read-only until you opt in, per item. It touches config files, and an agent that reorganizes your setup based on an inference you never saw is not a tool you want.

## Security

A single `SKILL.md`. No scripts, no network calls, no dependencies. Read it before you install it — advice that applies to every skill from every source, including this one.

It reads `~/.claude.json`, which can hold API keys for your MCP servers. The skill instructs the agent to read that file by structure and key names only, never printing credential values. Verify that on your first run.

## Tested on

Built by auditing real, drifted setups, not synthetic ones:

- **A large setup** — 6 plugins, ~80 injected skills, 14 overlapping design skills across two scopes, 4 competing session-start actors, no global CLAUDE.md
- **A lean setup** — 2 plugins, 1 local skill, no CLAUDE.md anywhere, orphaned telemetry from long-removed skills

Both macOS. Not yet tested on Windows, on team-shared project configs, or on setups with custom hooks. If you run it somewhere new, an issue reporting what it got wrong is the most useful thing you can contribute.

## Version history

- **1.0.0** — Initial release. Inventory reconciliation, usage telemetry as first-class evidence, gap analysis for what's missing, read-only by default.

## License

MIT.

Built with [Anthropic's Agent Skills format](https://code.claude.com/docs/en/skills).
