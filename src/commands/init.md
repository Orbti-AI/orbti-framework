---
name: orbit:init
description: Initialize ORBIT in a project with conversational setup
argument-hint:
allowed-tools: [Read, Write, Bash, Glob, AskUserQuestion]
---

<model>sonnet</model>

<objective>
Initialize the `.orbit/` structure in a project directory through conversational setup.

**When to use:** Starting a new project with ORBIT, or adding ORBIT to an existing codebase.

Creates PROJECT.md, STATE.md, and ROADMAP.md populated from conversation - user does not manually edit templates.
</objective>

<execution_context>
@~/.claude/orbit-framework/workflows/init-project.md
@~/.claude/orbit-framework/templates/PROJECT.md
@~/.claude/orbit-framework/templates/STATE.md
@~/.claude/orbit-framework/templates/ROADMAP.md
</execution_context>

<context>
Current directory state (check for existing .orbit/)
</context>

<process>
**Follow workflow: @~/.claude/orbit-framework/workflows/init-project.md**

The workflow implements conversational setup:

1. Check for existing .orbit/ (route to resume if exists)
2. Create directory structure
3. Ask: "What's the core value this project delivers?"
4. Ask: "What are you building?"
5. Confirm project name (infer from directory)
6. Populate PROJECT.md, ROADMAP.md, STATE.md from answers
7. Display ONE next action: `/orbit:refine`

**Key behaviors:**
- Ask ONE question at a time
- Wait for response before next question
- Populate files from answers (user doesn't edit templates)
- End with exactly ONE next action
- Build momentum into planning phase
</process>

<success_criteria>
- [ ] .orbit/ directory created
- [ ] PROJECT.md populated with core value and description from conversation
- [ ] STATE.md initialized with correct loop position
- [ ] ROADMAP.md initialized (phases TBD until planning)
- [ ] User presented with ONE clear next action
</success_criteria>
