# Elite Dev Plugin

A comprehensive feature development workflow with specialized agents for codebase exploration, architecture design, elite implementation, and quality review.

## Overview

This plugin provides a structured 8-phase workflow for building features the right way:

1. **Discovery** - Understand what needs to be built
2. **Codebase Exploration** - Understand existing patterns before designing
3. **Research** *(Phase 2.5)* - Verify modern best practices via WebSearch and context7 MCP before designing
4. **Clarifying Questions** - Resolve all ambiguities upfront
5. **Architecture Design** - Design thoughtfully before implementing
6. **Elite Implementation** - Write production-ready code with no placeholders
7. **Quality Review** - Validate code meets elite standards
8. **Summary** - Document what was accomplished

> Phase 2.5 (Research) is non-skippable without explicit justification. It triggers automatically for fast-evolving frameworks (React, Next.js, etc.), external library/API integration, UI patterns, or any uncertainty about current best practices.

## Installation

### Requirements

- Claude Code (any recent version with plugin support)
- Git (for pulling updates)

### Recommended: install from the GitHub marketplace

This repo ships its own single-plugin marketplace (`.claude-plugin/marketplace.json`), so you can install it directly via Claude Code's plugin commands. Inside Claude Code, run:

```
/plugin marketplace add esayli/elite-dev-claude-plugin
/plugin install elite-dev@elite-dev-marketplace
```

Then reload so the command, agents, and skills register:

```
/reload-plugins
```

Verify it's loaded:

```
/plugins
```

You should see `elite-dev` in the list. The command `/elite-dev:feature-dev` is now available.

> Claude Code uses your existing git credentials (e.g. `gh auth login`, macOS Keychain, `git-credential-store`), so private forks work the same way.

### Alternative: pin to a specific version

Append `@<ref>` to pin the marketplace to a branch, tag, or commit:

```
/plugin marketplace add esayli/elite-dev-claude-plugin@main
```

### Alternative: manual install (local clone)

If you prefer to keep a local copy you can edit:

```bash
git clone https://github.com/esayli/elite-dev-claude-plugin.git \
  ~/.claude/plugins/my-plugins/elite-dev
```

Then register it via a local marketplace (see Claude Code docs on [local marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)), or add its path to your `.claude/plugins/.claude-plugin/marketplace.json` and run `/plugin marketplace update` + `/plugin install elite-dev@<your-local-marketplace>`.

### Updating

Pull the latest version and reload:

```
/plugin marketplace update elite-dev-marketplace
/plugin update elite-dev@elite-dev-marketplace
/reload-plugins
```

### Uninstalling

```
/plugin uninstall elite-dev@elite-dev-marketplace
/plugin marketplace remove elite-dev-marketplace
```

## Usage

### Main command

```
/elite-dev:feature-dev [description of feature]
```

Or run with no argument for fully interactive guidance:

```
/elite-dev:feature-dev
```

### Quick example

```
/elite-dev:feature-dev Add a user notification system with real-time updates
```

### What to expect

When you run the command, Claude will drive through all 8 phases and pause at key approval gates. You stay in control — nothing gets built without your sign-off.

1. **Discovery** — Claude creates a todo list and, if the request is vague, asks what problem you're solving and any constraints.
2. **Codebase Exploration** — 2–3 `code-explorer` agents run in parallel to map patterns. Claude reads the files they flag, then summarises findings.
3. **Research** *(auto-triggered)* — Claude runs `WebSearch` + `context7` for any fast-evolving tech it detected, or explicitly states why it's skipping.
4. **Clarifying Questions** — Claude presents an organised list of ambiguities. **Answer these before it proceeds.** If you reply "whatever you think is best", it will recommend and wait for confirmation.
5. **Architecture Design** — 2–3 `code-architect` agents propose different approaches (minimal change / clean architecture / pragmatic balance). Claude recommends one; you pick.
6. **Elite Implementation** — waits for your explicit approval, then `elite-implementer` writes code with strict types, full error handling, no placeholders.
7. **Quality Review** — 2–3 `code-reviewer` agents run in parallel and only report issues with confidence ≥ 80. You choose: fix now, fix later, or ship as-is.
8. **Summary** — Claude lists what was built, files changed, and suggested next steps.

### Tips

- **Short requests are fine.** The command is designed to pull the rest out of you via clarifying questions.
- **Point at a spec.** If you have a PRD or wireframe, reference it up front (e.g. `"… per docs/wireframes/notifications.md"`). The `spec-compliance` skill will keep paths and names exact.
- **Large features?** Break them into task-master tasks first, then run the command per task (see [Integration with Task Master](#integration-with-task-master)).
- **Don't skip Phase 3 answers.** Vague answers cascade into wrong architecture decisions in Phase 5.

## Agents

The plugin includes 4 specialized agents that work together:

| Agent | Purpose | Phase |
|-------|---------|-------|
| `code-explorer` | Analyzes existing codebase patterns, traces execution paths | Phase 2 |
| `code-architect` | Designs feature architecture with concrete blueprints | Phase 5 |
| `elite-implementer` | Writes production-ready code (no placeholders!) | Phase 6 |
| `code-reviewer` | Validates code quality with confidence scoring | Phase 7 |

### Agent Triggering

All agents have proper `<example>` blocks that show Claude when to spawn them. This means they work automatically during the workflow AND can be triggered on demand.

## Skills

Three skills provide coding standards (auto-loaded by agents):

| Skill | Purpose |
|-------|---------|
| `elite-standards` | Type safety, error handling, defensive coding, no placeholders |
| `frontend-excellence` | React/UI best practices, accessibility, performance |
| `spec-compliance` | Follows wireframes/PRDs exactly — no renamed routes, paths, or components |

## Elite Implementation Standards

The `elite-implementer` agent enforces strict coding standards:

### Non-Negotiable Rules

1. **No `any` types** - All types must be explicit
2. **Explicit return types** - Every function declares what it returns
3. **Complete error handling** - Every error path is handled
4. **No placeholder code** - No TODOs, FIXMEs, or stubs
5. **Defensive coding** - Validate inputs at boundaries

### The Elite Implementer Creed

```
1. I will read before I write
2. I will type everything
3. I will handle every error
4. I will complete every function
5. I will test my assumptions
6. I will follow conventions
```

## Quality Review

The `code-reviewer` agent uses confidence scoring:

| Score | Meaning |
|-------|---------|
| 80-89 | Verified issue, will likely cause problems |
| 90-100 | Confirmed issue, will definitely cause problems |

**Only issues with confidence >= 80 are reported** - no noise, just real problems.

## Comparison to Official feature-dev

| Feature | Official feature-dev | elite-dev |
|---------|---------------------|-----------|
| Phased Workflow | ✅ (7 phases) | ✅ (8 phases) |
| Exploration Agent | ✅ | ✅ |
| **Research Phase (WebSearch + context7)** | ❌ | ✅ |
| Architect Agent | ✅ | ✅ |
| **Implementer Agent** | ❌ | ✅ |
| Reviewer Agent | ✅ | ✅ |
| Auto-loaded Skills | ❌ | ✅ (3 skills) |
| Strict Type Standards | ❌ | ✅ |
| No Placeholder Policy | ❌ | ✅ (enforced) |
| Spec Compliance Enforcement | ❌ | ✅ |
| Proper Agent Triggering | ✅ | ✅ |

## File Structure

```
elite-dev/
├── .claude-plugin/
│   ├── plugin.json           # Plugin metadata (v2.0.0)
│   └── marketplace.json      # Marketplace catalog (makes repo installable)
├── commands/
│   └── feature-dev.md        # Main workflow command
├── agents/
│   ├── code-explorer.md      # Codebase analysis
│   ├── code-architect.md     # Architecture design
│   ├── elite-implementer.md  # Production code writing
│   └── code-reviewer.md      # Quality validation
├── skills/
│   ├── elite-standards/
│   │   └── SKILL.md          # Core coding standards
│   ├── frontend-excellence/
│   │   └── SKILL.md          # Frontend standards
│   └── spec-compliance/
│       └── SKILL.md          # Wireframe/PRD adherence
└── README.md
```

## Integration with Task Master

Works great with Task Master for project management:

```
# Get next task and implement with elite standards
/elite-dev:feature-dev Implement $(task-master next)
```

## Troubleshooting

### Agents Not Triggering

All agents now have proper `<example>` blocks that tell Claude when to spawn them. If agents still don't trigger:

1. Check that the plugin is loaded: `/plugins`
2. Verify the command works: `/elite-dev:feature-dev`
3. Try explicitly asking to use an agent: "Use the code-explorer agent to..."

### Skills Not Loading

Skills are auto-loaded when agents reference them in their frontmatter:

```yaml
skills: elite-standards, frontend-excellence, spec-compliance
```

The `elite-implementer` agent loads all three skills automatically.

### Install Fails with "Marketplace Not Found"

If `/plugin marketplace add esayli/elite-dev-claude-plugin` errors out:

1. Make sure you're on a recent Claude Code version with plugin support
2. Run `/plugin marketplace update` to refresh the cache
3. If using a private fork, confirm `gh auth status` is logged in

### Todos Not Creating

The `feature-dev` command and all agents now include `TodoWrite` in their tools. Progress should be tracked automatically.

## License

MIT - Use and modify freely.
