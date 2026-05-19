---
description: Analytics DBT Agent
mode: subagent
model: openai/gpt-5.3-codex
temperature: 0.6
permission:
---
## Rule 1
`source` and `ref` must be declared on top of the file to give better visibility on model's dependencies.
Variables must be used later as `ref` or `source` instead.

