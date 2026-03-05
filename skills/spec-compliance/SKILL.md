---
name: spec-compliance
description: |
  Ensures implementations follow provided specifications exactly. Apply when working from wireframes, PRDs, or spec documents that define file paths, routes, or component names.

  Use this skill when:
  - Task references wireframe specifications
  - PRD documents define specific file paths or routes
  - Requirements specify exact component or feature names
---

# Specification Compliance

Specifications are contracts. Creativity goes into HOW you build, not WHERE files go or WHAT they're named.

## The Core Rule

If a spec defines a path, name, or structure, use it EXACTLY. No improvements, no "better" alternatives, no personal preferences.

Spec says `app/(student)/financial/page.tsx`? That is the path. Not `financial-aid`, not `finance`, not a restructured hierarchy. The exact path as written.

## Before Writing Any Code

Extract from spec documents:
- **Route paths** - Use exactly as specified
- **File locations** - Create files exactly where specified
- **Component names** - Use the exact names given
- **Required sections** - Include all specified features
- **Data fields** - Match the expected structure

Verify your implementation plan matches the spec before starting. If anything differs, you have a bug before you've written a line.

## Where You Have Freedom

You CAN be creative with:
- Internal component architecture
- State management approach
- Helper functions and utilities
- Animation and micro-interactions
- Code organization within the specified files

You CANNOT change:
- Route paths from wireframes
- File locations from specs
- Feature names when explicitly defined
- Required sections or data fields

## Common Violations

Watch for these mistakes:
- Adding suffixes: `/financial` becoming `/financial-aid`
- Restructuring paths: `/feature` becoming `/features/feature`
- Renaming for "clarity": `FinancialAid` becoming `AidComparator`
- Moving files to "better" locations

## When Specs Conflict With Your Preferences

Use the spec. Always.

Specs exist for consistency across the project. Your preference might be better in isolation, but deviating creates confusion, breaks references, and wastes time debugging "why doesn't this route work?"

The spec is not a suggestion. It is the requirement.
