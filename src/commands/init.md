---
name: orbti:init
description: Initialize ORBTI in a project with conversational setup
argument-hint:
allowed-tools: [Read, Write, Bash, Glob, AskUserQuestion]
---

<model>sonnet</model>

<objective>
Initialize the `.orbti/` structure in a project directory through conversational setup.

**When to use:** Starting a new project with ORBTI, or adding ORBTI to an existing codebase.

Creates PROJECT.md, STATE.md, and ROADMAP.md populated from conversation - user does not manually edit templates.
</objective>

<execution_context>
@~/.claude/orbti-framework/workflows/init-project.md
@~/.claude/orbti-framework/templates/PROJECT.md
@~/.claude/orbti-framework/templates/STATE.md
@~/.claude/orbti-framework/templates/ROADMAP.md
</execution_context>

<context>
Current directory state (check for existing .orbti/)
</context>

<process>
**Follow workflow: @~/.claude/orbti-framework/workflows/init-project.md**

The workflow implements conversational setup:

1. Check for existing .orbti/ (route to resume if exists)
2. Create directory structure
3. Ask: "What's the core value this project delivers?"
4. Ask: "What are you building?"
5. Confirm project name (infer from directory)
6. Populate PROJECT.md, ROADMAP.md, STATE.md from answers
7. Display ONE next action: `/orbti:refine`

**Key behaviors:**
- Ask ONE question at a time
- Wait for response before next question
- Populate files from answers (user doesn't edit templates)
- End with exactly ONE next action
- Build momentum into planning phase
</process>

<success_criteria>
- [ ] .orbti/ directory created
- [ ] PROJECT.md populated with core value and description from conversation
- [ ] STATE.md initialized with correct loop position
- [ ] ROADMAP.md initialized (phases TBD until planning)
- [ ] User presented with ONE clear next action
</success_criteria>
