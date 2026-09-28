# Skills

## 创建Skill

安装node, npx, Python

mkdir -p my-first-skill

Create my-first-skill/SKILL.md

```
---
name: my-first-skill
description: Turn rough meeting notes into a compact decision record when the user asks to capture a technical decision.
---

# Decision record

Extract the decision, context, alternatives, owner, and next review date.
If the notes do not contain a decision, ask one clarifying question instead
of inventing one.
```

&nbsp;

## Install the complete reviewer bundle

npx skills add rohitg00/ai-engineering-from-scratch --skill skill-contract-reviewer --full-depth

&nbsp;

## 运行Skill

|                   |                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------- |
| Host              | Explicit invocation                                                                   |
| Codex             | skill-contract-reviewer , or choose it from /skills , then provide the review request |
| Claude Code       | /skill-contract-reviewer followed by the review request                               |
| Portable fallback | Use skill-contract-reviewer to review the target package.                             |

&nbsp;

## 卸载Skill

npx skills remove skill-contract-reviewer

&nbsp;

## Skills encode procedural knowledge

An agent skill is a directory whose entry point is SKILL.md. The entry file contains YAML frontmatter followed by Markdown instructions. The directory can also contain references, scripts, and assets.

&nbsp;

## The portable core

The Agent Skills specification requires two frontmatter fields:

```
---
name: release-readiness
description: Inspect a release candidate when the user asks whether a version is ready to publish.
---
```

name is the stable identifier. It must satisfy the specification's naming rules and match the parent directory. description is both documentation and routing metadata. It should say what the skill does and when it applies.

The portable optional fields are:

|               |                                    |                                   |
| ------------- | ---------------------------------- | --------------------------------- |
| Field         | Purpose                            | Portability note                  |
| license       | State the terms for the package    | Core specification                |
| compatibility | State environmental requirements   | Core specification                |
| metadata      | Carry string-valued extension data | Core specification                |
| allowed-tools | Suggest pre-approved tools         | Experimental; host support varies |

The Markdown body holds the operational instructions. It should define the workflow, decision points, failure behavior, and direct paths to supporting resources.

```
# Release readiness

Use this workflow for a release candidate, not for ordinary development builds.

1. Read `references/release-policy.md`.
2. Run `python3 scripts/inspect_release.py --format json`.
3. Stop if the report contains a blocking failure.
4. Produce the checklist from `assets/release-checklist.md`.
5. Ask for approval before any publish or tag action.
```

&nbsp;

## Skill Discovery

A skill becomes useful before its body is loaded. Its name and description earn a place in the catalog; its deeper files earn context only when the task reaches them.

Scope is runtime policy：

|               |                        |                                |
| ------------- | ---------------------- | ------------------------------ |
| Scope         | Example root           | Intended ownership             |
| Workspace     | /.agents/skills/       | Project maintainers            |
| User          | /skills/               | One developer                  |
| Administrator | /skills/               | Machine or organization policy |
| Plugin        | A signed plugin bundle | Plugin publisher and installer |
| Built-in      | Runtime package        | Runtime vendor                 |

&nbsp;

## Three disclosure levels

- Level 1: catalog metadata
  - The model needs enough information to distinguish the skill from neighbors. The specification estimates roughly 100 tokens per catalog entry, but actual serialization and tokenization belong to the host.
- Level 2: active instructions
  - the body should function as a map and a procedure. The specification recommends keeping SKILL.md under 500 lines. That is a design signal, not a target to fill.
  - The body should contain:
    - the task boundary;
    - the default workflow;
    - branch conditions;
    - direct references to deeper files;
    - tool and script contracts;
    - failure and stopping behavior;
    - the expected output and its verification.
- Level 3: supporting resources
  - References supply prose or data. Scripts provide deterministic computation. Assets are copied, filled, or transformed into deliverables rather than treated as instructions.

|             |                  |                          |                                            |
| ----------- | ---------------- | ------------------------ | ------------------------------------------ |
| Directory   | Model reads it?  | Model executes it?       | Typical content                            |
| references/ | Yes, when needed | No                       | schemas, policies, domain guides           |
| scripts/    | May inspect it   | Through a permitted tool | validators, converters, collectors         |
| assets/     | Only if useful   | No                       | templates, fixtures, images, starter files |

&nbsp;

## 进一步阅读

- https://agentskills.io/specification
- https://agentskills.io/skill-creation/best-practices
- https://code.claude.com/docs/en/skills
