# Agent Skills and Reusable Procedures

## Learning objectives

Distinguish a skill from a tool, memory, and runtime; package a repeatable procedure in a portable directory; and review its instructions and scripts before enabling it.

## 1. What a skill does

A skill packages reusable task knowledge: when to use it, instructions to follow, examples, edge cases, and optionally reference files or scripts. It helps a model apply a procedure consistently. A skill does not itself guarantee correct output, grant authority, or execute a tool unless the surrounding host/runtime provides that capability.

Think of the pieces this way:

- **Skill:** task-specific instructions and supporting material.
- **Tool:** an executable capability with inputs, outputs, and permissions.
- **Memory/state:** information retained for a conversation, user, or workflow.
- **Workflow/graph:** explicit control flow among steps and tools.
- **Harness:** the runtime and policies that supply context, execute actions, preserve state, and observe results.

## 2. A portable skill directory

The Agent Skills specification describes a directory containing a required `SKILL.md` and optional `scripts/`, `references/`, and `assets/` folders. `SKILL.md` begins with YAML frontmatter; `name` and `description` are required. The name uses lowercase letters, digits, and hyphens; the description should state what the skill does and when to use it.

```text
skills/
└── csv-quality-audit/
    ├── SKILL.md
    ├── references/
    │   └── quality-checks.md
    └── scripts/
        └── profile_csv.py
```

```markdown
---
name: csv-quality-audit
description: Profile a CSV schema, missing values, duplicates, and suspicious ranges. Use when auditing a new tabular dataset before modeling.
---

# CSV quality audit

1. Confirm the file's source, license, and row meaning.
2. Inspect its shape, columns, types, nulls, duplicates, and value ranges.
3. Report findings without deleting or rewriting the original data.
4. Ask before applying a cleaning rule that could remove valid observations.

See references/quality-checks.md for the full checklist.
```

The format allows optional metadata such as license/compatibility and an experimental `allowed-tools` field; host support varies. Runtime permission policy must remain outside the skill text.

## 3. Progressive disclosure

Keep the description specific enough for a host to decide when the skill is relevant. Put the core procedure in `SKILL.md`; move long reference material to linked files and detailed code to scripts. This lets a host reveal a skill only when useful instead of placing every procedure in every prompt.

Do not create a skill for a one-off instruction. Good candidates are repeated, bounded jobs with known inputs, steps, expected artifacts, and common edge cases: data audit, paper review, camera calibration, experiment report, or incident triage.

## 4. Make the procedure verifiable

A strong skill defines the required inputs, ordered steps, expected outputs, validation checks, stop/ask conditions, and known failure cases. For code-generation or analysis skills, specify how a result is checked. A skill should say when evidence is insufficient and how to report uncertainty.

## 5. Security review

Skills are executable supply-chain inputs when they reference scripts or tools. Read their instructions and inspect scripts before activation. Check network access, file access, dependencies, secrets, and side effects. Do not trust a skill because it is well-formatted or downloaded from a repository. Give referenced tools only the permissions needed for the task.

## Practice

Write a skill for “analyze a model's false negatives.” Include when it applies, files/inputs it needs, a step-by-step error taxonomy, output format, privacy constraints, and a rule that prevents the agent from changing the test set or model without approval.

## Common mistakes

- Writing a vague skill description that matches too many tasks or none.
- Putting hundreds of pages into the main instruction file instead of loading references as needed.
- Treating a skill as a sandbox, safety policy, or persistent memory system.
- Running an included script before reviewing its permissions and behavior.

## Further reading

- [Agent Skills specification](https://agentskills.io/specification)
- [LangChain skills documentation](https://docs.langchain.com/oss/python/deepagents/skills)
