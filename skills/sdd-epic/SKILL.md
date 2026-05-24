---
name: sdd-epic
description: Generate Epic folder structure
---
# SDD Epic Skill

Given the project context and existing specifications,
identify major units of work.

Generate epics that:

- have clear business or technical goals
- can be completed independently
- expose measurable acceptance criteria
- can be decomposed into tasks

Output:
specs/
└── epics/
   └── EPIC-[EPIC_NUMBER]-[EPIC_NAME]/
      ├── epic.md
      ├── tasks/
      │   ├── [TASK-NAME]-[TASK_NUMBER].md
      │   ├── [TASK-NAME]-[TASK_NUMBER].md
      │   └── [TASK-NAME]-[TASK_NUMBER].md
      ├── handoff.json
      └── worktrees/
          ├── [TASK-NAME]-[TASK_NUMBER]/
          ├── [TASK-NAME]-[TASK_NUMBER]/
          └── [TASK-NAME]-[TASK_NUMBER]/
