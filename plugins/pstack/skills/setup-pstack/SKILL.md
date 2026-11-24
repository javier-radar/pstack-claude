---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available models and efforts and writes a user rule in ~/.claude/rules/ that overrides the skill defaults. Use for /pstack:setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write `~/.claude/rules/pstack-models.md`, a user rule that every new session loads, to set pstack's model and effort per role.

## Steps

### 1. Detect available models

Enumerate the values the `Agent` tool's `model` parameter accepts in this session, and the levels its `effort` parameter accepts. The tool's schema lists both, and it is the dependable source. If you cannot detect any, ask the user to paste the models and efforts they have access to. Never write a model or effort you have not confirmed is available. The aliases `inherit-parent` and `auto` are always valid even though they are not detected models.

### 2. Load current state

The default role mapping is the rule shape shown in step 5 below. If `~/.claude/rules/pstack-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `hardest tasks`, `architect runners`, `reflect tooling`, or `why investigators`, is from a retired role. Drop it.

A rule without the `# defaults: 3` line comes from an older setup. That setup wrote a line for every role, so some of its values are old defaults, not choices. Before step 3, replace these values:

- A `# budget: unlimited` line becomes `large`, because no role runs above `xhigh`.
- An old value is one that names `sonnet`. An old value in `feature, refactoring`, `bug-fix`, `perf-issue`, `hillclimb`, or `swarm workers` takes the role's step 5 default. In a panel role (`arena runners`, `arena cross-judge pool`, `interrogate reviewers`), a list of exactly one `opus` entry and one old value, in either order, takes the role's step 5 list.

Every other value is the user's choice and stays, including an old value in any other role and every `inherit-parent` or `auto` value. Tell the user which values you replaced.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer `AskUserQuestion` over free text. Offer these three options with these exact labels, and name the current budget when the rule records one. With no rule, say that `large` matches the skill defaults.

- `large (xhigh reasoning)`
- `medium (high reasoning)`
- `small (medium reasoning)`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role you changed by model, list, or alias (`inherit-parent`, `auto`). A role value is `<model> <effort>`, and the effort token is the last token. Each role has a target effort, which is its default's effort moved by the budget. `large` keeps it, `medium` lowers it one step, and `small` lowers it two steps, on the ladder `xhigh` > `high` > `medium` > `low`. No role runs above `xhigh`, and a target at the bottom of the ladder stays there. Most defaults are at `xhigh`, and each budget label names where those land. Set the effort token of every real value in the role, panel entries included, to the role's target. If that effort is not in the detected set, use the highest detected effort at or below the target. If the model is not in the detected set, mark the role as needing a choice. `inherit-parent` and `auto` do not change. So `medium` turns `opus xhigh` into `opus high` and `opus high` into `opus medium`, and `small` turns them into `opus medium` and `opus low`. The `sonnet` entry in a panel role takes its own target, `xhigh` moved by the budget, so `large` keeps `sonnet xhigh`, `medium` writes `sonnet high`, and `small` writes `sonnet medium`. A model that does not take the written effort runs at its nearest level when the role spawns.

**(c) Show the roles and confirm.** Show every role with its value, marking any model not in the detected set as needing a choice. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering the detected models at `xhigh` or below plus `inherit-parent` and `auto` as the options. Both aliases mean the role runs on the parent session's model. Prefer `AskUserQuestion` over free text. For panel roles (arena runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena runs one judge. It tries only values naming a model other than the parent's, in list order, so order it by preference. An `inherit-parent` or `auto` value never judges. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every real value written must name a detected model and a detected effort at `xhigh` or below. `inherit-parent` and `auto` always pass. If a chosen model or effort is not available, stop and ask again.

### 5. Write the rule

Write `~/.claude/rules/pstack-models.md` with no frontmatter, the comment lines below, the `# defaults: 3` line, a `# budget` line with the chosen label and the effort that label names, and one `<model> <effort>` line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Keep the role lines in this file only. Do not copy them into `CLAUDE.md`. Shape:

```
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# Each value is `<model> <effort>`. Pass the model as the `Agent` call's `model` and the last token as its `effort`.
# A value of `inherit-parent` or `auto` runs the role on the parent session's model (omit `model` on the `Agent` call and spawn `general-purpose`). Alias entries in a panel list still count toward its fan-out.
# defaults: 3
# budget: large (xhigh)
feature, refactoring: opus xhigh
bug-fix: opus xhigh
perf-issue: opus xhigh
hillclimb: opus xhigh
judgment and prose: opus xhigh
arena runners: fable high, opus high, sonnet xhigh
arena cross-judge pool: opus high, sonnet xhigh
swarm workers: opus high
interrogate reviewers: fable high, opus high, sonnet xhigh
```

### 6. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a project `verify` or `verify-*` skill under `.claude/skills/`, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /pstack:create-verification-skill." On yes, read and follow the [**create-verification-skill**](../create-verification-skill/SKILL.md) skill. On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /pstack:create-formal-verification-skill." On yes, read and follow the [**create-formal-verification-skill**](../create-formal-verification-skill/SKILL.md) skill. On no, move on without pushing.
