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

## Usage

### Main Command

```
/elite-dev:feature-dev [description of feature]
```

Or just:
```
/elite-dev:feature-dev
```
For interactive guidance through the workflow.

### Example

```
/elite-dev:feature-dev Add a user notification system with real-time updates
```

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
│   └── plugin.json           # Plugin metadata (v2.0.0)
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
skills: elite-standards, frontend-excellence
```

The `elite-implementer` agent loads both skills automatically.

### Todos Not Creating

The `feature-dev` command and all agents now include `TodoWrite` in their tools. Progress should be tracked automatically.

## License

MIT - Use and modify freely.
