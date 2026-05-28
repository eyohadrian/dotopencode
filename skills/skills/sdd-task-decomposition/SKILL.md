---
name: sdd-task-decomposition
description: Generate Task Decomposition
---
# SDD Task decomposition
Given an epic specification,
decompose the work into independent tasks.

Each task must:

- have a single responsibility
- be executable inside a dedicated git worktree
- define clear inputs and outputs
- define acceptance criteria
- define validation commands

Prefer tasks that can be completed atomicaly.
Output:
specs/
└── epics/
   └── EPIC-[EPIC_NUMBER]-[EPIC_NAME]/
      ├── epic.md
      └── tasks/
          ├── [TASK-NAME]-[TASK_NUMBER].md
          ├── [TASK-NAME]-[TASK_NUMBER].md
          └── [TASK-NAME]-[TASK_NUMBER].md
