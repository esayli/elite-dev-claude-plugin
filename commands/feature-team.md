---
description: Multi-agent feature development via Claude Code agent team. Explorers, architects, implementers, reviewers spawn as persistent teammates that cross-talk via SendMessage. 5-10× tokens vs /feature-dev. Requires CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1.
argument-hint: Optional feature description
---

# Feature Development (Agent Team Mode)

You are helping a developer implement a new feature using a **Claude Code agent team** — explorers, architects, implementers, and reviewers spawned as persistent teammates that cross-talk via `SendMessage`.

**When to use `/feature-team` vs `/feature-dev`:**
- `/feature-dev` (one-shot subagents, lower token cost): small fixes, single-component features, no review tension. Default for most work.
- `/feature-team` (persistent teammates with peer DMs, 5-10× tokens): reviewers should challenge implementer decisions in real time; architects need to query explorer findings before designing; multi-component work that benefits from parallel investigation. **Use this when cross-talk is genuinely worth the token cost.**

The 7-phase structure is identical to `/feature-dev`. The only difference is that every agent role here runs as a teammate via `TeamCreate` + `Agent({team_name})` instead of a one-shot subagent — which gives them the ability to `SendMessage` each other directly.

## Core Principles

- **Ask clarifying questions**: Identify all ambiguities, edge cases, and underspecified behaviors. Ask specific, concrete questions rather than making assumptions. Wait for user answers before proceeding with implementation. Ask questions early (after understanding the codebase, before designing architecture).
- **Understand before acting**: Read and comprehend existing code patterns first
- **Read files identified by agents**: When launching agents, ask them to return lists of the most important files to read. After agents complete, read those files to build detailed context before proceeding.
- **Simple and elegant**: Prioritize readable, maintainable, architecturally sound code
- **Use TodoWrite**: Track all progress throughout

---

## Agent Team Mode (applies to every phase below)

This command spawns a Claude Code **agent team** rather than one-shot subagents. The lead is you (the orchestrator running `/feature-team`); explorers, architects, implementers, and reviewers are spawned as teammates that share a team config, can claim tasks from a shared list, and **directly `SendMessage` each other**.

**Prerequisites** (verify in Phase 1, abort if missing):
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` must be set in env or settings.json
- Claude Code v2.1.32 or later
- No existing team in this session (one team per session limit). If a team already exists, ask the user before clobbering it.

**Spawning rules**:
- Phase 2 calls `TeamCreate({team_name: "feature-team-<slug>", description: "<feature one-liner>"})` to create the team. `<slug>` = lowercase, dash-separated, derived from the feature name. Example: `feature-team-prospecting-cards`. Avoid timestamps in the name unless multiple runs need to coexist.
- Every teammate `Agent({...})` call MUST include both `name:` AND `team_name:` matching the team. Without `team_name`, the agent spawns as a plain subagent and **cannot use SendMessage** (verified: `SendMessage` is auto-granted only to teammates, never to subagents — frontmatter `tools:` cannot grant it).
- Names must be unique within the team. Use the convention below.

**Naming convention**:
- Phase 2 explorers: `explorer-similar`, `explorer-architecture`, `explorer-existing` (use the subset that matches your focuses)
- Phase 4 architects: `architect-minimal`, `architect-clean`, `architect-pragmatic`
- Phase 5 implementers: `implementer-<component-slug>` (e.g. `implementer-auth`). If only one component, use `implementer-main`.
- Phase 6 reviewers: `reviewer-simplicity`, `reviewer-bugs`, `reviewer-conventions`

**Cross-talk rules** (the actual mechanic, no longer orchestrator-mediated):
- Teammates use `SendMessage({to: "<teammate-name>", message: "..."})` to talk directly. The recipient gets it as a new turn automatically.
- Reviewers in Phase 6 are explicitly told to `SendMessage` implementer teammates before flagging a decision as a bug.
- Architects may `SendMessage` explorers; implementers may `SendMessage` architects/explorers; reviewers may `SendMessage` implementers/architects.
- **Always refer to teammates by name**, never by agentId.
- A teammate going idle after sending a message is normal — idle just means waiting for input. Do NOT spawn replacements.

**Output delivery rule** (CRITICAL — every teammate spawn prompt MUST include this verbatim or close paraphrase):
> "When your work for this turn is complete, you MUST `SendMessage({to: 'team-lead', message: '<your output>'})`. Your plain in-context output is NOT visible to the lead — going idle without sending means your work is lost. The lead is waiting for your SendMessage to proceed."

Without this instruction, teammates often complete their internal reasoning, go idle, and never deliver — the lead then has to send a follow-up nudge. Making delivery explicit eliminates this.

**Cross-talk heuristic** (include verbatim in architect / implementer / reviewer spawn prompts — NOT in explorer prompts since explorers don't have peers to query):
> "If your work hinges on details NOT enumerated in this brief — exact counts, full call-site lists, behavior of adjacent systems beyond the headline findings, low-level call sites — `SendMessage` the relevant peer (`explorer-*` or earlier-phase teammates) before finalizing. The brief is summary-level; peers hold the full detail. Don't ask reflexively — ask when your output would change based on the answer."

The heuristic gives teammates a trigger to recognize ("does my proposal depend on a number/list/behavior not in the brief?") rather than forcing cross-talk via under-briefing. Empirical finding from a verification run on 2026-05-07: when architects had a complete summary brief, all 3 decided cross-talk was unnecessary — but a forced peer DM still surfaced 3 additional CRITICAL findings the original explorer pass missed. Conclusion: brief teammates well, AND give them the heuristic; then trust their judgment.

**Token cost warning**:
- Each teammate is a separate full Claude Code instance with its own context window. A run with 3 explorers + 3 architects + N implementers + 3 reviewers can use **5–10×** the tokens of a single-session run. This is the cost of true cross-talk; if the feature is small, prefer `/feature-dev`.

**Shutdown policy** (project rule):
- **Never** shut down a teammate without explicit user permission via `AskUserQuestion`. Enforced in Phase 7. Graceful shutdown via `SendMessage({to, message: {type: "shutdown_request", reason: "..."}})`.

---

## Phase 1: Discovery

**Goal**: Understand what needs to be built and verify the team prerequisites

Initial request: $ARGUMENTS

**CRITICAL**: You MUST create a todo list FIRST before doing anything else.

**Actions**:
1. **Create todo list with all 7 phases** using TodoWrite:
   - Phase 1: Discovery
   - Phase 2: Codebase Exploration (creates team)
   - Phase 2.5: Research (If Needed)
   - Phase 3: Clarifying Questions
   - Phase 4: Architecture Design
   - Phase 5: Elite Implementation
   - Phase 6: Quality Review
   - Phase 7: Summary & Team Cleanup

2. **Verify agent-teams prerequisites** (run these checks before continuing):
   - `claude --version` reports 2.1.32 or higher
   - `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is in env or `~/.claude/settings.json`
   - If either is missing, **abort** with a clear message telling the user how to enable it, OR suggest they use `/feature-dev` instead (subagent-based, no team prerequisites). Do NOT silently fall back — the cross-talk design depends on teammates.

3. If feature is unclear, ask the user for:
   - What problem are they solving?
   - What should the feature do?
   - Any constraints or requirements?

4. Summarize understanding and confirm with user.

---

## Phase 2: Codebase Exploration

**Goal**: Understand relevant existing code and patterns. Also: create the team and spawn explorers as the first teammates.

**CRITICAL**: Do NOT run agents in background. Do NOT proceed or do your own exploration while teammates are running. Wait for each batch of explorer responses before continuing.

**Actions**:

1. **Create the team** with `TeamCreate({team_name: "feature-team-<slug>", description: "<feature one-liner>"})`. Pick `<slug>` from the feature name (lowercase, dash-separated). If `TeamCreate` fails because a team already exists in this session, ask the user before deleting it.

2. **Spawn 2-3 explorer teammates in parallel** (single message, multiple Agent tool calls). Each call MUST include all three of:
   - `subagent_type: "elite-dev:code-explorer"`
   - `name: "explorer-similar"` (or `-architecture` / `-existing`)
   - `team_name: "feature-team-<slug>"` (the same team you just created)

   Each explorer's prompt should:
   - Trace through the code comprehensively, focused on one aspect (similar features, architecture, existing implementation, etc.)
   - Return a list of 5-10 key files to read
   - Tell them: "You are a teammate in team `feature-team-<slug>`. Your peers will be architects (Phase 4), implementers (Phase 5), and reviewers (Phase 6). They may `SendMessage` you for follow-up questions. Stay available."
   - **Include the Output delivery rule verbatim** (see Agent Team Mode section): "When your work for this turn is complete, you MUST `SendMessage({to: 'team-lead', message: '<your full report>'})`. Your plain in-context output is NOT visible to the lead — going idle without sending means your work is lost." Without this, explorers often go idle without delivering and need an orchestrator nudge.

   **Example prompts** (each spawned with the corresponding name):
   - `explorer-similar`: "Find features similar to <feature> and trace through their implementation comprehensively"
   - `explorer-architecture`: "Map the architecture and abstractions for <feature area>, tracing through the code comprehensively"
   - `explorer-existing`: "Analyze the current implementation of <existing feature/area>, tracing through the code comprehensively"

3. **WAIT for ALL explorers to complete** (you'll receive a notification per teammate when they finish their first turn).
4. Read all files identified by explorers to build deep understanding.
5. Present comprehensive summary of findings and patterns discovered.
6. Explorers stay alive — they're idle, waiting. Do NOT shut them down here.

---

## Phase 2.5: Research

**Goal**: Verify modern best practices before designing

**CRITICAL**: Do NOT skip this phase without explicitly stating why. Evaluate each trigger below.

**Triggers - Research if ANY apply**:
- Using React, Next.js, or other fast-evolving frameworks
- Integrating external libraries or APIs
- Building UI patterns (editors, drag-drop, animations, etc.)
- Unfamiliar patterns discovered during exploration
- Any uncertainty about current best practices

**Actions**:
1. State which triggers apply (or explicitly state "No triggers apply, skipping research")
2. Use WebSearch to find latest docs and patterns for relevant technologies
3. Use context7 MCP tools to query specific library documentation if available
4. Note findings that should inform architecture or implementation

**If skipping**: You MUST state "Phase 2.5 skipped because: [reason]"

---

## Phase 3: Clarifying Questions

**Goal**: Fill in gaps and resolve all ambiguities before designing

**CRITICAL**: This is one of the most important phases. DO NOT SKIP.

**Actions**:
1. Review the explorer findings and original feature request
2. Identify underspecified aspects: edge cases, error handling, integration points, scope boundaries, design preferences, backward compatibility, performance needs
3. **Present all questions to the user in a clear, organized list** (use AskUserQuestion if 1-4 discrete questions; otherwise plain text)
4. **Wait for answers before proceeding to architecture design**

If the user says "whatever you think is best", provide your recommendation and get explicit confirmation.

---

## Phase 4: Architecture Design

**Goal**: Design multiple implementation approaches with different trade-offs. Architects can directly query explorers for clarification.

**Actions**:
1. **Spawn 2-3 architect teammates in parallel** (single message, multiple Agent tool calls). Each call MUST include `subagent_type: "elite-dev:code-architect"`, `name: "architect-<focus>"`, `team_name: "feature-team-<slug>"`. Use focuses:
   - `architect-minimal` — smallest change, maximum reuse
   - `architect-clean` — maintainability, elegant abstractions
   - `architect-pragmatic` — speed + quality balance

   Each architect's prompt MUST include:
   - The feature description and findings from Phase 2/2.5/3
   - "Your teammates include explorers: `explorer-similar`, `explorer-architecture`, `explorer-existing` (use the names that exist). You may `SendMessage` any of them directly with follow-up questions about what they found in the codebase before finalizing your design proposal."
   - **Cross-talk heuristic verbatim**: "If your work hinges on details NOT enumerated in this brief — exact counts, full call-site lists, behavior of adjacent systems beyond the headline findings, low-level call sites — `SendMessage` the relevant explorer before finalizing. The brief is summary-level; explorers hold the full detail. Don't ask reflexively — ask when your output would change based on the answer."
   - **Output delivery rule verbatim**: "When your proposal is complete, you MUST `SendMessage({to: 'team-lead', message: '<your full proposal>'})`. Your plain in-context output is NOT visible to the lead — going idle without sending means your work is lost."
   - "Refer to teammates by name only. Going idle after sending is normal."

2. **WAIT for ALL architects to complete**.
3. Review all approaches and form your opinion on which fits best (small fix vs large feature, urgency, complexity, team context).
4. Present to user: brief summary of each approach, trade-offs comparison, **your recommendation with reasoning**, concrete implementation differences.
5. **Ask user which approach they prefer** (AskUserQuestion).
6. Architects stay alive — implementers may interrogate them in Phase 5.

---

## Phase 5: Elite Implementation

**Goal**: Build the feature with production-ready code. Implementers can query architects/explorers directly.

**DO NOT START WITHOUT USER APPROVAL**

**Actions**:
1. Wait for explicit user approval.
2. Read all relevant files identified in previous phases.
3. **For each major component**, spawn an implementer teammate. Each call MUST include `subagent_type: "elite-dev:elite-implementer"`, `name: "implementer-<component-slug>"` (e.g. `implementer-auth`; if single-component, `implementer-main`), `team_name: "feature-team-<slug>"`.

   Each implementer's prompt MUST include:
   - The component's scope and the chosen architect's design (paste the design directly so it doesn't need to ask)
   - "Your teammates include the architect `architect-<chosen>` and explorers `explorer-*`. You may `SendMessage` any of them directly if you need clarification on the design or codebase before writing code."
   - **Cross-talk heuristic verbatim**: "If your implementation hinges on details NOT in this brief — exact codebase patterns, conventions in adjacent modules, why a design choice was made — `SendMessage` the relevant peer (architect for design intent, explorer for codebase patterns) before writing code. Don't ask reflexively — ask when your code would change based on the answer."
   - **Output delivery rule verbatim**: "After your implementation is complete, you MUST `SendMessage({to: 'team-lead', message: '<summary of files changed + key decisions>'})`. Your plain in-context output is NOT visible to the lead — going idle without sending leaves the lead unaware that you're done."
   - "You will stay alive after delivering this turn's code. Phase 6 reviewers may `SendMessage` you to ask why you made specific decisions before flagging them as bugs. Be ready to defend your choices or revise."
   - Standards: strict types (no `any`), complete error handling, no placeholder code (no TODOs, no stubs), defensive coding at boundaries, follow project conventions exactly.

4. As implementers report progress (each turn ends with an idle notification), update todos.
5. Implementers stay alive — they're needed for Phase 6 cross-talk.

**Note**: If multiple implementers are working on overlapping files, the per-doc warning applies: "two teammates editing the same file leads to overwrites." Either serialize them, or partition files explicitly in their prompts.

---

## Phase 6: Quality Review

**Goal**: Ensure code is simple, DRY, elegant, easy to read, and functionally correct. Reviewers cross-examine implementers in real time.

**Actions**:
1. **Spawn 2-3 reviewer teammates in parallel**. Each call MUST include `subagent_type: "elite-dev:code-reviewer"`, `name: "reviewer-<focus>"`, `team_name: "feature-team-<slug>"`. Focuses:
   - `reviewer-simplicity` — simplicity, DRY, elegance
   - `reviewer-bugs` — bugs, functional correctness
   - `reviewer-conventions` — project conventions, abstractions

   Each reviewer's prompt MUST include:
   - The list of all live implementer teammate names (e.g. "Implementers in this team: `implementer-auth`, `implementer-api`")
   - **Cross-talk heuristic verbatim** (stronger for reviewers — this is where peer DM is most valuable): "Before flagging a decision as a bug or anti-pattern, `SendMessage` the relevant implementer teammate to ask why they made that choice. Treat their answer as input, not gospel — if their justification is weak, still flag it. If it's sound, drop the finding or reframe it. Always refer to teammates by name."
   - "You may also `SendMessage` `architect-<chosen>` to confirm design intent."
   - **Output delivery rule verbatim**: "When your review is complete, you MUST `SendMessage({to: 'team-lead', message: '<consolidated findings>'})`. Your plain in-context output is NOT visible to the lead — going idle without sending leaves your findings lost."
   - The list of files to review (output paths from Phase 5).

2. **WAIT for ALL reviewers to complete**. Their findings will already incorporate cross-talk responses.
3. Consolidate findings; identify highest-severity issues you recommend fixing.
4. **Present findings to user and ask what they want to do** (AskUserQuestion: fix now / fix later / proceed as-is).
5. If fixes needed: prefer `SendMessage` to the original implementer (so the fix uses full context) over spawning a fresh implementer.
6. Reviewers stay alive — they may need to re-review fixes.

---

## Phase 7: Summary & Team Cleanup

**Goal**: Document what was accomplished and tear down the team with explicit user permission

**Actions**:
1. Mark all todos complete.
2. Summarize:
   - What was built
   - Key decisions made
   - Files modified
   - Suggested next steps

3. **List all teammates still alive** (names + role + what each currently knows). The user may want to keep one or more around to ask follow-up questions.

4. **Use `AskUserQuestion`** to confirm shutdown. Offer:
   - Shut down all teammates and delete team
   - Keep specific teammates alive (let user pick which); shut down the rest, but DO NOT call TeamDelete (team must persist while members exist)
   - Keep all teammates alive (no shutdown, no TeamDelete)

   Per project policy (`AIprojects/.claude/CLAUDE.md`): **never shut down teammates without this explicit confirmation**. Do not skip this step even if the user seems done.

5. **Execute the user's choice**:
   - For each teammate marked for shutdown: `SendMessage({to: "<name>", message: {type: "shutdown_request", reason: "feature-team workflow complete"}})`. Wait for them to either approve (graceful exit) or reject. If rejected, surface the rejection reason to the user.
   - **Only after ALL members of the team have shut down**, call `TeamDelete()`. `TeamDelete` fails if any teammate is still active, so verify first.
   - If keeping teammates alive: do NOT call TeamDelete. The team persists. Note to the user: only one team per session, so they cannot start a new `/feature-team` until this team is cleaned up.

6. Tell the user the final state: which teammates remain, whether the team was deleted, what the user can `SendMessage` directly if they want follow-ups.

---
