# superpowers-custom

A fork of **[obra/superpowers](https://github.com/obra/superpowers)** (v6.0.3 base),
packaged as a standalone Claude Code plugin. It is a **drop-in replacement** for the
official `superpowers` plugin — you install this and disable the official one, so there is
never a duplicate or a conflicting copy of any skill.

## Why a fork instead of a patch?

The customizations live in three skills. Earlier this was distributed as a patch applied
into the official plugin's cache — but Claude Code wipes that cache on every superpowers
update, silently reverting the skills to stock (which, for `subagent-driven-development`,
means subagents start committing again — the exact behavior the fork removes).

A separate plugin lives **outside** the superpowers cache, so an update can't touch it.
Nothing to reapply, nothing to detect, no drift. The cost is that you merge upstream
yourself when you want it (`git merge upstream`, conflicts only in the 3 custom skills).

## What's different from upstream

- **subagent-driven-development — no-commit model:** implementer/fix subagents NEVER
  commit; the **controller** commits each task after its review passes and a
  file-collision check. Per-task review uses git **tree snapshots** (`scripts/snapshot` +
  `review-package` reworked to diff tree-ish, with `-- <paths>` to scope one task of a
  parallel batch). No commit list — the work is uncommitted by design.
- **Disjoint-file parallel subagents:** tasks with non-overlapping files run as concurrent
  background/non-blocking subagents; reviewed per file-set, committed per task.
- **Model policy:** Sonnet implementers for step-based plans; escalate to the
  most-capable "advisor" model only when the **plan itself** is defective.
- **finishing-a-development-branch:** PR body built from the repo's PR template and passed
  via `--body-file`; after opening, monitor CI + Copilot + Code-Quality comments.
- **using-git-worktrees:** GitHub **issue → linked branch** entry points.

Everything else tracks upstream. Skills are namespaced `superpowers-custom:` (e.g.
`superpowers-custom:subagent-driven-development`).

## Install (Claude Code)

```text
/plugin marketplace add stanlrt/superpowers-custom
/plugin install superpowers-custom@superpowers-custom
```

Then **disable the official superpowers plugin** so only this one is active (otherwise
both load and the model sees two copies of every skill):

- `/plugin` → manage → disable `superpowers@claude-plugins-official`

Verify: open a new session and check the skill listing shows
`superpowers-custom:subagent-driven-development` and **not** `superpowers:…`.

> The official and the fork cannot both be enabled — they provide the same skill set under
> different namespaces. Pick one. To go back, re-enable the official plugin and disable this.

## Updating the fork from upstream

```bash
git remote add upstream https://github.com/obra/superpowers   # once
git fetch upstream --tags
git merge upstream/main        # or a specific tag, e.g. v6.1.0
# resolve conflicts (only in the 3 custom skills), then bump version in
# .claude-plugin/plugin.json + marketplace.json, commit, push
```

See [CLAUDE.md](CLAUDE.md) for conflict-resolution rules.

## Credit

Methodology and skills © Jesse Vincent / the superpowers authors, MIT-licensed. This is a
personal fork and is not affiliated with or supported by the upstream project.
