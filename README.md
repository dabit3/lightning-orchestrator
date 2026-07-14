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
5. Applies trivial fixes directly or resumes the same executor with targeted feedback when a correction is needed.
6. Reports the completed changes, verification results, and any remaining risks.

The skill uses one executor by default to minimize context duplication and cost. For clear, low-risk tasks the preflight reads are issued in the same round as the dispatch, review reads are batched the same way, and long-running suites run in the background during diff review — keeping orchestrator overhead off the critical path. Purely explanatory or read-only requests are handled directly when delegation would not add value.

Three further optimizations keep handoff overhead in check:

- **Trivial edits skip delegation.** Every handoff has a roughly fixed coordination cost — the work order and report are each written by one side and read by the other — so a few clearly understood, low-risk lines are edited directly instead of being routed through an executor.
- **Follow-up work resumes the same executor.** Resuming keeps the executor's accumulated repository context and prompt cache warm, so later work orders skip rediscovery instead of re-paying a cold start.
- **Long exploratory tasks get steering checkpoints.** Instead of one open-ended order, the work is split into milestone-scoped resumes with a review between each, so unproductive directions are caught early rather than after a long run.

When fanning out to parallel executors, the orchestrator embeds its already-established shared context (build commands, conventions, key paths) into every work order, since parallel workers cannot see each other's discoveries and would otherwise repeat the same research.

## Requirements

- Devin with support for project skills and custom subagent profiles
- Access to the `swe-1.7-lightning` model

## Installation

Install both the skill and its executor profile. They must be installed together because the skill explicitly delegates implementation to `lightning-executor`.

### Skills CLI

Install the skill for Devin in the current project:

```bash
npx skills add dabit3/lightning-orchestrator --agent devin
```

The Skills CLI installs `SKILL.md` packages, but not Devin-specific custom agent profiles. Install the required executor separately:

```bash
mkdir -p .devin/agents/lightning-executor
curl -fsSL \
  https://raw.githubusercontent.com/dabit3/lightning-orchestrator/main/.devin/agents/lightning-executor/AGENT.md \
  -o .devin/agents/lightning-executor/AGENT.md
```

For a global installation, add `--global` and place the executor in Devin's global configuration directory:

```bash
npx skills add dabit3/lightning-orchestrator --agent devin --global
mkdir -p ~/.config/devin/agents/lightning-executor
curl -fsSL \
  https://raw.githubusercontent.com/dabit3/lightning-orchestrator/main/.devin/agents/lightning-executor/AGENT.md \
  -o ~/.config/devin/agents/lightning-executor/AGENT.md
```

### Manual installation

If you already cloned this repository, copy both directories into your project:

```bash
mkdir -p .devin/skills .devin/agents
cp -R /path/to/lightning-orchestrator/.devin/skills/lightning .devin/skills/
cp -R /path/to/lightning-orchestrator/.devin/agents/lightning-executor .devin/agents/
```

For a manual global installation, copy them into your user configuration directory:

```bash
mkdir -p ~/.config/devin/skills ~/.config/devin/agents
cp -R /path/to/lightning-orchestrator/.devin/skills/lightning ~/.config/devin/skills/
cp -R /path/to/lightning-orchestrator/.devin/agents/lightning-executor ~/.config/devin/agents/
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

## Permissions

Both definitions pre-approve the read-only inspection commands on the critical path — `git status`, `git diff`, and `git log` — so fresh sessions do not stall on approval prompts just to inspect repository state. Writes, edits, and all other commands follow your normal approval flow.

To remove per-edit approval prompts entirely — and to let background fan-out executors edit files without a foreground resume — opt in by adding `edit` to the executor's allowed permissions in `.devin/agents/lightning-executor/AGENT.md`:

```yaml
permissions:
  allow:
    - Exec(git status)
    - Exec(git diff)
    - Exec(git log)
    - edit
```

Only enable this if you are comfortable with the executor editing files without prompting.

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
