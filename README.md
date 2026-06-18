# superpowers-custom

My customized `skills/` for [obra/**superpowers**](https://github.com/obra/superpowers) (v6.0.2). Drop-in replacement for the plugin's `skills/` directory.

## What's different from upstream

- **subagent-driven-development — no-commit model:** implementer/fix subagents NEVER commit; the **controller** commits each task after its review passes and after a file-collision check.
- **Snapshot-based review (no commits needed):** per-task diff via git tree snapshots (`scripts/snapshot` + `review-package` reworked to diff tree-ish, with `-- <paths>` to scope one task of a parallel batch).
- **Disjoint-file parallel subagents:** tasks with non-overlapping files run as concurrent **background / non-blocking** subagents in one shared worktree.
- **Model policy:** Sonnet implementers for step-based plans; escalate to the most-capable "advisor" model only when the **plan itself** is defective.
- **finishing-a-development-branch:** PR body built from the repo's PR template and passed via `--body-file` (never inline `--body`); after opening, monitor CI + Copilot + Code-Quality comments and resolve them (Code-Quality via the GitHub GraphQL API).
- **using-git-worktrees:** GitHub **issue → linked branch** entry points (start from chat → write spec to a new issue → `gh issue develop`; or start from an existing issue link).

## Install

Replace the plugin's `skills/` directory with this repo's `skills/`. Find your installed version first (the `<ver>` folder).

**macOS / Linux**
```bash
SP=~/.claude/plugins/cache/claude-plugins-official/superpowers
VER=$(ls "$SP" | grep -E '^[0-9]' | sort -V | tail -1)
mv "$SP/$VER/skills" "$SP/$VER/skills.bak"          # back up upstream
git clone https://github.com/stanlrt/superpowers-custom /tmp/spc
cp -r /tmp/spc/skills "$SP/$VER/skills"
chmod +x "$SP/$VER/skills/subagent-driven-development/scripts/"*
```

**Windows (PowerShell)**
```powershell
$SP = "$env:USERPROFILE\.claude\plugins\cache\claude-plugins-official\superpowers"
$VER = (Get-ChildItem $SP -Directory | Where-Object Name -match '^\d' | Sort-Object Name)[-1].Name
Move-Item "$SP\$VER\skills" "$SP\$VER\skills.bak"   # back up upstream
git clone https://github.com/stanlrt/superpowers-custom $env:TEMP\spc
Copy-Item "$env:TEMP\spc\skills" "$SP\$VER\skills" -Recurse
```

> Plugin updates overwrite the cache — re-run the install after each superpowers update. Built on v6.0.2; if a newer version changed these skills, diff before replacing. MIT, same as upstream.
