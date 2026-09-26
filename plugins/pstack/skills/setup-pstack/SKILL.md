---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available models and writes the pstack model roles into omp's config. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

pstack routes every delegated job to one of nine role agents, and each agent reads its model from one `modelRoles` alias in `~/.omp/agent/config.yml`. This skill writes those nine aliases. The chat model stays the operator's choice, made with `/model`, and pstack never overrides it. The operator can change any alias later in `/model`'s Roles view without rerunning this skill.

| Agent | `modelRoles` alias | Jobs | Capability to bind |
|---|---|---|---|
| `pstack-code` | `pstack_code` | feature, refactoring, bug-fix, perf-issue, hillclimb, how explorer, why investigators, swarm workers | your fast code model |
| `pstack-judgment` | `pstack_judgment` | judgment and prose, how explainer, why synthesizer | your strongest judgment model |
| `pstack-hardest` | `pstack_hardest` | hardest tasks, reflect judgment, reflect synthesizer | your strongest reasoning model |
| `pstack-panel-1` | `pstack_panel_1` | interrogate reviewer A, arena and architect runner 1 | a strong reasoning model, family 1 |
| `pstack-panel-2` | `pstack_panel_2` | interrogate reviewer B, arena and architect runner 2 | a strong reasoning model, family 2 |
| `pstack-panel-3` | `pstack_panel_3` | interrogate reviewer C, arena and architect runner 3 | a strong reasoning model, family 3 |
| `pstack-cross-judge` | `pstack_cross_judge` | arena cross-judge | a strong model on a different family from the chat model |
| `pstack-reflect-divergent` | `pstack_reflect_divergent` | reflect divergent | a different model family from `pstack_hardest` |
| `pstack-reflect-tooling` | `pstack_reflect_tooling` | reflect tooling | your strongest instruction-following model |

An unset alias falls back to the agent's second model entry, `@task` for `pstack-code` and `@default` for the rest, so an unconfigured install still runs.

## Steps

### 1. Detect available models

Run `omp models` to list the models configured on this machine and group them by provider family. Never write a selector you have not seen in that list.

### 2. Load current state

Read `modelRoles` in `~/.omp/agent/config.yml` and treat any existing `pstack_*` entries as the current choices. A `task.agentModelOverrides` entry for a `pstack-*` agent beats its alias, so list any you find and ask whether to remove them.

### 3. Budget, map, and confirm

**(a) Ask for a budget** with `ask`, naming the current one when the entries carry a suffix.

- `unlimited — max reasoning`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Propose a model per alias** from the detected models, following the capability column. Put the three panel seats on three different models, on different families where the machine offers them, and keep `pstack_reflect_divergent` off the family of `pstack_hardest`.

**(c) Show the table and confirm** with `ask`. Offer to accept or change specific aliases, using the detected models as options.

### 4. Validate

Every selector written must be in the step 1 list. The budget becomes an effort suffix, `<selector>:<effort>`. When a model does not offer that effort, take its highest effort at or below the target and say so.

### 5. Write the roles

Write the nine `pstack_*` entries into `modelRoles` in `~/.omp/agent/config.yml`, replacing any earlier `pstack_*` entries so reruns stay idempotent, and leave every other role alone. `skill://omp-mechanics` holds the file shape. To keep a fallback when a model's quota runs out, add a `retry.fallbackChains` entry keyed by the same alias name.

### 6. Confirm

Tell the user which entries were written, that they apply to new sessions and new spawns, and that `/model`'s Roles view changes any of them later.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill`. On no, move on without pushing.
