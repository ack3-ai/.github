# ack3 Git and agent workflow

This is the canonical Git workflow for every repository in the `ack3-ai`
GitHub organization. It applies regardless of operating system, editor, coding
agent, clone location, or directory layout. Contributors may organize local
checkouts however they prefer; no parent workspace repository or common folder
structure is required.

Repository-specific `AGENTS.md`, `CLAUDE.md`, and `CONTRIBUTING.md`
instructions also apply and take precedence when stricter.

## Non-negotiable rules

- Determine the current repository's Git boundary before modifying files or Git
  state. Never assume a parent or sibling directory belongs to the same
  repository.
- Never do routine work directly on a default or release branch. Changes enter
  those branches through a pull request.
- One task has one stable branch. When multiple tasks or agents run in parallel,
  give each writer a separate worktree or clone; its filesystem location is a
  local choice.
- Do not intentionally run simultaneous writers on one branch or worktree.
  Git's non-fast-forward push rejection is the safety net if two machines
  overlap; never bypass it with a force push.
- The remote task branch is the cross-machine synchronization record. Pull
  requests are for review and merging, not routine agent coordination.
- If a checkout contains changes you did not create, do not stage, stash,
  discard, switch, or reformat them. Leave it in place and create a clean
  worktree or clone.
- A change spanning repositories gets one branch and pull request per affected
  repository, using the same work ID and slug.

## Starting or resuming work

Inspect the target repository before editing:

```sh
git fetch origin --prune
git status --short --branch
git symbolic-ref --short refs/remotes/origin/HEAD
git worktree list
```

First check whether the task already has a remote branch. If it does, create or
reuse a clean checkout tracking that branch and verify local `HEAD` matches the
remote before editing.

For a new task, choose the base branch required by the repository's own
instructions, or use `origin/HEAD`, then create and publish a task branch. For
example, when using a worktree:

```sh
git worktree add <local-worktree-path> \
  -b <type>/<work-id>-<slug> origin/<base>
cd <local-worktree-path>
git push -u origin HEAD
```

Pushing the branch immediately makes the task visible from another machine. Do
not create an empty commit merely to reserve it.

- Claude Code: `claude --worktree <unique-local-name>` provides isolation.
  Confirm the generated branch follows this policy.
- Codex: managed worktrees may start on a detached `HEAD`. Create or switch to
  the stable task branch and set its upstream before the first commit.
- Manual sessions: use any local checkout layout that preserves one writer per
  branch and avoids modifying someone else's worktree.

## Branch names

Use:

```text
<type>/<work-id>-<short-slug>
```

- `type`: `feat`, `fix`, `docs`, `content`, `perf`, `refactor`, `test`, `build`,
  `ci`, `chore`, or `hotfix`.
- `work-id`: the issue or ticket number; if none exists, use `YYYYMMDD`.
- `short-slug`: a lowercase kebab-case description of one outcome.

Examples:

```text
feat/123-request-router
docs/20260812-git-policy
```

Do not include a person, machine, tool, model, agent, or session identity in a
branch name. Local worktree directory names may include a local-only suffix.

## Continuous synchronization and commits

Every agent turn that modifies files is a potential machine switch. Before
yielding control, the agent must:

1. stage only files owned by the task;
2. run the relevant checks;
3. commit the coherent current state;
4. push it to the task branch; and
5. report anything that could not be committed or pushed.

Incomplete but coherent work may use a `wip: checkpoint ...` commit. Never
include known-broken, secret, generated, or unrelated files merely to create a
checkpoint. Structure work so a mutating turn normally ends at a coherent
checkpoint.

This policy authorizes agents to create and push checkpoint commits to the
current task branch without asking after each one. It does not authorize direct
pushes to default or release branches, force pushes, merges, or unrelated
files.

- Keep every commit an isolated, explainable change. Separate unrelated work.
- Use `type(scope): imperative summary` when a scope is useful, for example
  `fix(router): preserve versioned docs paths`.
- Stage explicit paths or patches. With unrelated changes present, never use
  `git add -A`, `git add .`, or blanket formatting.
- Before committing, run `git diff --check`, inspect `git diff --staged`, and
  run the smallest relevant test or validation suite.
- Never commit secrets, local environment files, caches, logs, or generated
  output unless the repository explicitly tracks that output.

If a push is rejected because the remote advanced, stop editing, fetch, inspect
the competing commits, and reconcile deliberately. Never force-push or
overwrite published task-branch history. Ask for a human decision only when the
competing changes cannot be reconciled safely.

## Pull requests

Open a pull request when the change is ready for review. Do not open a draft
merely to claim a branch or publish agent status. Keep one pull request focused
on one outcome.

The description should contain only what a reviewer needs:

- the problem and intended outcome;
- changed areas and deliberate exclusions;
- tests or checks run and their results;
- screenshots or generated artifacts when output is visual;
- material risks, migrations, rollout, or rollback notes; and
- linked issues and cross-repository pull requests, including merge order.

Do not use PR comments as a heartbeat or append a handoff log after every
checkpoint; pushed commits already update the pull request. Request human
attention only when a complete change is ready or a decision is required.

Use squash merge by default so checkpoint commits remain useful during
development without cluttering the long-lived branch. Delete the remote task
branch after merge.

## GitHub enforcement

Organization and repository rulesets provide server-side safeguards for
protected branches. Their configuration in GitHub is the source of truth, and
repository-specific controls may be stricter. GitHub cannot observe local
worktrees or unpushed edits, so checkout isolation and checkpoint pushes remain
agent responsibilities.

## Recovery and cleanup

- Do not use `git reset --hard`, `git clean -fd`, destructive checkout or
  restore, or branch/worktree deletion on changes whose ownership is uncertain.
- Do not use a stash as cross-machine synchronization. Never pop or drop a
  stash you did not create.
- If the current checkout is dirty, preserve it and branch from the clean
  remote ref in another worktree or clone.
- After merge, confirm the pull request is complete, remove only your clean
  task checkout, prune worktree metadata if applicable, and delete your local
  task branch.

## Basis

This policy follows the official guidance for
[GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow),
[organization rulesets](https://docs.github.com/en/organizations/managing-organization-settings/creating-rulesets-for-repositories-in-your-organization),
[Git worktrees](https://git-scm.com/docs/git-worktree),
[Codex worktrees](https://learn.chatgpt.com/docs/environments/git-worktrees),
and [Claude Code worktrees](https://code.claude.com/docs/en/worktrees).
