---
name: Handoff
description: Context sharing among agents
---

Write a handoff document summarising the current conversation so a fresh agent can continue the work. Save to the temporary directory of the user's OS - not the current workspace.

Include a "suggested skills" section in the document, naming which skills the next agent should call the Skill tool for.

Do not duplicate content already captured in other artifacts (specs, plans, ADRs, issues, commits, diffs). Reference them by path or URL instead.

Redact any sensitive information, such as API keys, passwords, or personally identifiable information.

If the user passed arguments, treat them as a description of what the next session will focus on and tailor the doc accordingly.

The template of the handoff would be like this: 

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
