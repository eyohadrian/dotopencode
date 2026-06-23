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
      ├── handoff.json
      ├── epic.md
      └── tasks/
          ├── [TASK_NUMBER]-[TASK-NAME]/
          │   └── [TASK_NUMBER]-[TASK-NAME]-wt/
          └── [TASK_NUMBER]-[TASK-NAME]/
          │   ├── README.md
          │   └── [TASK_NUMBER]-[TASK-NAME]-wt/
          ├── [TASK_NUMBER]-[TASK-NAME]/
          │   ├── README.md
          │   └── [TASK_NUMBER]-[TASK-NAME]-wt/
          └── [TASK_NUMBER]-[TASK-NAME]/
              ├── README.md
              └── [TASK_NUMBER]-[TASK-NAME]-wt/

## Worktrees

Each task worktree directory must either be a real git worktree. 
The reason of this is to isolate and paralelize the tasks implementation by other `AGENTS`. 
