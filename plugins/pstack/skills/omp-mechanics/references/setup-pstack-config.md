# The omp shape of pstack's model configuration

Amends **setup-pstack** steps 2, 5, and 6. The table in the skill maps jobs to agents. This file is
the file format.

Each of the nine `pstack-*` agents names one alias in its frontmatter, for example
`model: ["@pstack_code", "@task"]`. omp expands the first entry through `modelRoles` in
`~/.omp/agent/config.yml`. An unset alias falls through to the second entry, so an unconfigured
install runs on `@task` for code and `@default` for everything else. A bare unset alias with no
fallback fails the spawn with "No model selected", which is why every agent carries two entries.

A `modelRoles` value is a selector `omp models` printed, optionally with an effort suffix
`<selector>:<effort>`, which is where the step 3 budget answer lands.

```yaml
modelRoles:
  # One line per pstack role agent. Each value is a selector `omp models` printed on this
  # machine plus the budget's effort suffix. Delete a line to fall back to @task or @default.
  pstack_code: <fast code model>:<effort>
  pstack_judgment: <strongest judgment model>:<effort>
  pstack_hardest: <strongest reasoning model>:<effort>
  pstack_panel_1: <reasoning model, family 1>:<effort>
  pstack_panel_2: <reasoning model, family 2>:<effort>
  pstack_panel_3: <reasoning model, family 3>:<effort>
  pstack_cross_judge: <model off the chat model's family>:<effort>
  pstack_reflect_divergent: <model off pstack_hardest's family>:<effort>
  pstack_reflect_tooling: <strongest instruction-following model>:<effort>
retry:
  fallbackChains:
    # Optional. Keyed by the alias name. A spawn through @pstack_code retries down this chain
    # when its model fails or runs out of quota.
    pstack_code:
      - <second-choice fast code model>
```

`/model`'s Roles view edits the same entries, so the operator can rebind one role without rerunning
the skill. `task.agentModelOverrides` keyed by a `pstack-*` agent name beats the alias. Setup does
not write it, and step 2 flags any it finds.

A retry fallback can move a panel seat onto another family. Tell the operator that a seat can land
off its configured family when its chain fires, and record the model each seat actually ran on.

Step 6 reports which entries were written and that they apply to new sessions and new spawns.
