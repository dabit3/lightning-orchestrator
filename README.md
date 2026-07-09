# Lightning

**Fast execution, smart orchestration for Devin.**

Lightning is a software-engineering skill that pairs the intelligence of your active Devin model with the speed and cost efficiency of SWE-1.7 Lightning. The active model plans, makes decisions, and reviews the result, while a dedicated Lightning subagent performs the implementation.

## Why Lightning?

Using one model for every step can be unnecessarily slow or expensive. Lightning separates the work into two roles:

| Role | Responsibility |
| --- | --- |
| Active Devin model | Understands the request, resolves important decisions, creates the work order, and reviews the result |
| `lightning-executor` | Inspects the code, implements the change, runs focused checks, and reports evidence |

This provides a fast implementation path without giving up higher-level reasoning or independent review.

## How it works

When you invoke the skill, the orchestrator:

1. Frames the objective, acceptance criteria, constraints, and validation needs.
2. Inspects enough of the repository to resolve material product or architecture questions.
3. Sends a self-contained work order to the `lightning-executor` subagent.
4. Reviews the resulting diff and verification evidence against the original request.
5. Resumes the same executor with targeted feedback if a correction is needed.
6. Reports the completed changes, verification results, and any remaining risks.

The skill uses one executor by default to minimize context duplication and cost. Purely explanatory or read-only requests are handled directly when delegation would not add value.

## Requirements

- Devin with support for project skills and custom subagent profiles
- Access to the `swe-1.7-lightning` model

## Installation

Install both the skill and its executor profile. They must be installed together because the skill explicitly delegates implementation to `lightning-executor`.

### Project installation

Copy the directories into the project where you want to use Lightning:

```bash
mkdir -p .devin/skills .devin/agents
cp -R /path/to/devin-orchestrator/.devin/skills/lightning .devin/skills/
cp -R /path/to/devin-orchestrator/.devin/agents/lightning-executor .devin/agents/
```

Project-scoped configuration can be committed and shared with the rest of the repository.

### Global installation

To make Lightning available in every project, install it in your user configuration directory:

```bash
mkdir -p ~/.config/devin/skills ~/.config/devin/agents
cp -R /path/to/devin-orchestrator/.devin/skills/lightning ~/.config/devin/skills/
cp -R /path/to/devin-orchestrator/.devin/agents/lightning-executor ~/.config/devin/agents/
```

Start a new Devin session after installation so the skill and agent profile are discovered.

## Usage

Invoke Lightning with a concrete software-engineering task:

```text
/lightning add pagination to the users endpoint and update its tests
```

```text
/lightning reproduce and fix the checkout total rounding bug
```

```text
/lightning refactor the cache adapter without changing its public API
```

Good requests describe the observable outcome and any important constraints. Lightning discovers routine implementation details and local repository conventions on its own.

## Behavior and safeguards

Lightning is designed to:

- prioritize correctness and safety over speed or cost
- preserve existing user changes
- implement the smallest coherent diff
- follow repository-specific instructions and conventions
- add or update focused tests when appropriate
- verify the result before reporting success
- avoid destructive operations and external side effects without explicit approval
- avoid unrelated refactors, dependencies, generated files, and documentation

If the custom executor profile is unavailable, Lightning stops and reports the missing profile rather than silently switching to another model.

## Repository structure

```text
.devin/
├── agents/
│   └── lightning-executor/
│       └── AGENT.md
└── skills/
    └── lightning/
        └── SKILL.md
```

- `.devin/skills/lightning/SKILL.md` defines the orchestration workflow.
- `.devin/agents/lightning-executor/AGENT.md` defines the model-pinned implementation agent.

## Customization

You can adapt the workflow by editing the two definitions:

- Change orchestration policy, review requirements, or delegation criteria in `SKILL.md`.
- Change the executor model, tool restrictions, implementation protocol, or permissions in `AGENT.md`.

Keep the skill's referenced profile name and the executor's `name` field aligned if you rename `lightning-executor`.
