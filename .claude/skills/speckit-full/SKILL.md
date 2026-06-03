---
name: speckit-full
description: Full speckit pipeline that runs specify → plan → tasks → implement in sequence for a given feature description. Use this whenever the user wants to go from a feature idea all the way to working code in one command. Trigger on phrases like "전체 파이프라인 실행", "처음부터 구현까지", "speckit-full", or when the user gives a feature description and wants everything done automatically.
---

# Speckit Full Pipeline

Runs the complete speckit workflow end-to-end:
1. `/speckit-specify` — write the feature specification
2. `/speckit-plan` — generate the implementation plan and design artifacts
3. `/speckit-tasks` — generate the task checklist
4. `/speckit-implement` — execute all tasks

## Input

The user's message after `/speckit-full` is the **feature description**. Pass it verbatim to `/speckit-specify`.

## Execution Rules

- **Strictly sequential**: never start the next step until the current step has fully completed and reported success.
- **Pause for user input**: if any step asks the user a question (e.g., clarification questions from speckit-specify), stop and wait. Resume only after the user has responded and the current step finishes.
- **Stop on failure**: if any step reports an error or cannot proceed, halt and tell the user what failed before continuing.

## Step-by-Step Instructions

### Step 1 — Specify

Invoke the skill:

```
/speckit-specify <feature description>
```

Wait for it to complete. If it asks clarification questions (marked with `[NEEDS CLARIFICATION]` or presents a Q1/Q2/Q3 table), pause here and wait for the user to answer before moving on.

Once speckit-specify reports "Readiness for the next phase", proceed to Step 2.

### Step 2 — Plan

Invoke the skill:

```
/speckit-plan
```

Wait for it to report completion and list the generated artifacts (plan.md, research.md, data-model.md, etc.). Then proceed to Step 3.

### Step 3 — Tasks

Invoke the skill:

```
/speckit-tasks
```

Wait for it to report the total task count and the path to tasks.md. Then proceed to Step 4.

### Step 4 — Implement

Invoke the skill:

```
/speckit-implement
```

Wait for it to complete all tasks and report final status.

## Completion

After all four steps finish, summarize:
- Feature directory created
- Key artifacts generated (spec.md, plan.md, tasks.md)
- Number of tasks completed
- Any remaining manual steps (e.g., git commit hooks that were skipped)
