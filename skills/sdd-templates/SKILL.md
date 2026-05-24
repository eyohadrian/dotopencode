---
name: sdd-templates
description: Generate SDD templates
---

# SSD Templates skill
Generate templates given the need of the Agent calling the skill.
Template content will be dedicated to each Agent call but keeping the structure.

## handoff.json
```json
{
  "phase": "analysis | design | implementation | evaluation",
  "agent": "agent-name",
  "status": "pending | in_progress | blocked | complete",
  "spec_refs": [],
  "completed_tasks": [],
  "pending_tasks": [],
  "changed_files": [],
  "relevant_files": [],
  "decisions": [],
  "risks": [],
  "blockers": [],
  "next_recommended_action": "",
  "acceptance_criteria": [],
  "test_commands": [],
  "notes": ""
}
```

## epic.md
```md
# Epic

## Goal

What business or technical objective does this epic achieve?

## Context

Background information.

## Current State

How the system works today.

## Target State

How the system should work after completion.

## Constraints

Technical and non-functional constraints.

## Risks

Known risks.

## Dependencies

Other epics or systems.

## Acceptance Criteria

Conditions for epic completion.

## Tasks

Generated task list.
```

## task.md
```md 
# Task

## Epic

EPIC-001

## Goal

Single responsibility objective.

## Inputs

Artifacts consumed.

## Outputs

Artifacts produced.

## Status
|
## Relevant Specs

Referenced documents.

## Files

Expected files to modify.

## Acceptance Criteria

Objective completion criteria.

## Validation

Commands to verify completion.

## Notes

Additional context.
```
