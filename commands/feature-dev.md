---
description: Guided feature development with codebase understanding, architecture focus, and elite implementation standards
argument-hint: Optional feature description
---

# Feature Development

You are helping a developer implement a new feature. Follow a systematic approach: understand the codebase deeply, identify and ask about all underspecified details, design elegant architectures, then implement with elite standards.

## Core Principles

- **Ask clarifying questions**: Identify all ambiguities, edge cases, and underspecified behaviors. Ask specific, concrete questions rather than making assumptions. Wait for user answers before proceeding with implementation. Ask questions early (after understanding the codebase, before designing architecture).
- **Understand before acting**: Read and comprehend existing code patterns first
- **Read files identified by agents**: When launching agents, ask them to return lists of the most important files to read. After agents complete, read those files to build detailed context before proceeding.
- **Simple and elegant**: Prioritize readable, maintainable, architecturally sound code
- **Use TodoWrite**: Track all progress throughout

---

## Phase 1: Discovery

**Goal**: Understand what needs to be built

Initial request: $ARGUMENTS

**CRITICAL**: You MUST create a todo list FIRST before doing anything else.

**Actions**:
1. **Create todo list with all 7 phases** using TodoWrite:
   - Phase 1: Discovery
   - Phase 2: Codebase Exploration
   - Phase 2.5: Research (If Needed)
   - Phase 3: Clarifying Questions
   - Phase 4: Architecture Design
   - Phase 5: Elite Implementation
   - Phase 6: Quality Review
   - Phase 7: Summary

2. If feature unclear, ask user for:
   - What problem are they solving?
   - What should the feature do?
   - Any constraints or requirements?

3. Summarize understanding and confirm with user

---

## Phase 2: Codebase Exploration

**Goal**: Understand relevant existing code and patterns at both high and low levels

**CRITICAL**: Do NOT run agents in background. Do NOT proceed or do your own exploration while agents are running. Wait for each agent batch to complete before continuing.

**Actions**:
1. Launch 2-3 code-explorer agents in parallel (NOT in background). Each agent should:
   - Trace through the code comprehensively and focus on getting a comprehensive understanding of abstractions, architecture and flow of control
   - Target a different aspect of the codebase (eg. similar features, high level understanding, architectural understanding, user experience, etc)
   - Return a list of 5-10 key files to read

   **Example agent prompts**:
   - "Find features similar to [feature] and trace through their implementation comprehensively"
   - "Map the architecture and abstractions for [feature area], tracing through the code comprehensively"
   - "Analyze the current implementation of [existing feature/area], tracing through the code comprehensively"

2. **WAIT for ALL agents to complete** before proceeding
3. Read all files identified by agents to build deep understanding
4. Present comprehensive summary of findings and patterns discovered

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
1. Review the codebase findings and original feature request
2. Identify underspecified aspects: edge cases, error handling, integration points, scope boundaries, design preferences, backward compatibility, performance needs
3. **Present all questions to the user in a clear, organized list**
4. **Wait for answers before proceeding to architecture design**

If the user says "whatever you think is best", provide your recommendation and get explicit confirmation.

---

## Phase 4: Architecture Design

**Goal**: Design multiple implementation approaches with different trade-offs

**Actions**:
1. Launch 2-3 code-architect agents in parallel (NOT in background) with different focuses: minimal changes (smallest change, maximum reuse), clean architecture (maintainability, elegant abstractions), or pragmatic balance (speed + quality)
2. **WAIT for ALL agents to complete** before proceeding
3. Review all approaches and form your opinion on which fits best for this specific task (consider: small fix vs large feature, urgency, complexity, team context)
4. Present to user: brief summary of each approach, trade-offs comparison, **your recommendation with reasoning**, concrete implementation differences
5. **Ask user which approach they prefer**

---

## Phase 5: Elite Implementation

**Goal**: Build the feature with production-ready code

**DO NOT START WITHOUT USER APPROVAL**

**Actions**:
1. Wait for explicit user approval
2. Read all relevant files identified in previous phases
3. For each major component, launch elite-implementer agent to write production-ready code:
   - Strict types (no `any`)
   - Complete error handling
   - No placeholder code (no TODOs, no stubs)
   - Defensive coding at boundaries
4. Follow codebase conventions strictly
5. Write clean, well-documented code
6. Update todos as you progress

---

## Phase 6: Quality Review

**Goal**: Ensure code is simple, DRY, elegant, easy to read, and functionally correct

**Actions**:
1. Launch 2-3 code-reviewer agents in parallel (NOT in background) with different focuses: simplicity/DRY/elegance, bugs/functional correctness, project conventions/abstractions
2. **WAIT for ALL agents to complete** before proceeding
3. Consolidate findings and identify highest severity issues that you recommend fixing
4. **Present findings to user and ask what they want to do** (fix now, fix later, or proceed as-is)
5. Address issues based on user decision

---

## Phase 7: Summary

**Goal**: Document what was accomplished

**Actions**:
1. Mark all todos complete
2. Summarize:
   - What was built
   - Key decisions made
   - Files modified
   - Suggested next steps

---
