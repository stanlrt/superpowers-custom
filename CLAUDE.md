# superpowers-custom — fork maintenance notes

This repo is a **fork of [obra/superpowers](https://github.com/obra/superpowers)**,
packaged as a standalone Claude Code plugin named `superpowers-custom`. It is a drop-in
replacement for the official `superpowers` plugin: install this, disable the official one.

## What diverges from upstream

Only three skills carry custom behavior (everything else tracks upstream verbatim):

- **subagent-driven-development** — no-commit model: implementer/fix subagents NEVER
  commit; the controller commits each task after its review passes and a file-collision
  check. Per-task review via git tree snapshots (`scripts/snapshot` + `review-package`
  diffing tree-ish, `-- <paths>` scoping). Disjoint-file parallel subagents run in the
  background. Sonnet implementers; advisor only when the plan itself is defective.
- **finishing-a-development-branch** — PR body from the repo's PR template via
  `--body-file`; after opening, monitor CI + Copilot + Code-Quality comments.
- **using-git-worktrees** — GitHub issue → linked branch entry points.

The plugin namespace is `superpowers-custom:` (not `superpowers:`); all internal skill
cross-references were rewritten to match.

## Merging upstream updates

This fork keeps the **full** upstream tree so updates are a normal git merge, not a patch.

```bash
git remote add upstream https://github.com/obra/superpowers   # once
git fetch upstream --tags
git merge upstream/main          # or the tag you want, e.g. v6.1.0
```

Conflicts should land **only** in the three customized skills above. When resolving:
- Keep the no-commit semantics in `subagent-driven-development` (subagents never commit).
- Keep `scripts/snapshot` and `scripts/review-package` resolving their scratch dir via
  the upstream `scripts/sdd-workspace` helper (`<repo-root>/.superpowers/sdd/`), NOT the
  old `<git-dir>/sdd/` — Claude Code denies agent writes under `.git/`.
- After merging, re-run the namespace rewrite if upstream added new `superpowers:` refs:
  `grep -rIoE 'superpowers:[a-z-]+' skills hooks` should return nothing; if it does,
  `sed -i 's/superpowers:/superpowers-custom:/g'` the offending files.
- Bump `version` in `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`
  to the new upstream base.

## Not affiliated with upstream

Original methodology and skills © Jesse Vincent / the superpowers authors (MIT). This is a
personal fork; do not file its issues or PRs against obra/superpowers.
