# Continuity Ledger

## Snapshot
- 2026-05-05 [USER] Goal: put all installed Codex skills into `/Users/federico.dambrosio@igenius.ai/Repos/skills-repo`.
- 2026-05-05 [TOOL] Source interpreted as local installed skills tree: `/Users/federico.dambrosio@igenius.ai/.codex/skills`.
- 2026-05-05 [TOOL] Destination is repo-local `skills/`.
- 2026-05-05 [TOOL] Repository directory is not a git repository (`git status` returned fatal: not a git repository).
- 2026-05-05 [ASSUMPTION] Plugin cache skills are not being vendored unless requested; current sync target is `$CODEX_HOME/skills`.
- 2026-05-05 [TOOL] Sync completed: all 64 non-`.DS_Store` files from `$CODEX_HOME/skills` are present under repo `skills/`.
- 2026-05-05 [USER] Goal expanded: include skills installed from plugins.
- 2026-05-05 [TOOL] Plugin sync completed from GitHub and Superpowers plugin caches; repo now has 28 `SKILL.md` files.
- 2026-05-05 [TOOL] Verification found no missing local, GitHub plugin, or Superpowers plugin files in `skills/`.
- 2026-05-05 [USER] New goal: create a skill for internal `domyn-swarm` CLI at `/Users/federico.dambrosio@igenius.ai/Repos/domyn-swarm`.
- 2026-05-05 [CODE] Added `skills/deploying-domyn-swarm/` for deploying/validating/monitoring vLLM clusters from Domyn Swarm YAML configs.

## Decisions
- 2026-05-05 [ASSUMPTION] D001 SUPERSEDED by D002: Initially treated "installed skills" as `$CODEX_HOME/skills`, excluding OS metadata such as `.DS_Store`.
- 2026-05-05 [USER] D002 ACTIVE: Include local `$CODEX_HOME/skills` plus plugin cached skills from enabled plugin caches.
- 2026-05-05 [ASSUMPTION] D003 ACTIVE: Copy plugin skill directories directly into repo `skills/` using source skill directory names.
- 2026-05-05 [ASSUMPTION] D004 ACTIVE: Preserve copied upstream skill scripts as-is even when over 300 LOC, because this task is vendoring installed skill contents rather than refactoring their internals.
- 2026-05-05 [CODE] D005 ACTIVE: Name the new Domyn Swarm skill `deploying-domyn-swarm`.
- 2026-05-05 [ASSUMPTION] D006 ACTIVE: Do direct pressure-scenario validation instead of subagent validation because the current runtime only allows spawning subagents when the user explicitly requests delegation.

## Done (recent)
- 2026-05-05 [TOOL] Confirmed `CONTINUITY.md` did not exist and initialized this ledger.
- 2026-05-05 [TOOL] Ran `rsync -a --exclude '.DS_Store' ~/.codex/skills/ skills/`.
- 2026-05-05 [TOOL] Verified `rsync -aicn --exclude '.DS_Store' ~/.codex/skills/ skills/` produced no pending copy/update output.
- 2026-05-05 [TOOL] Ran plugin syncs from `/Users/federico.dambrosio@igenius.ai/.codex/plugins/cache/openai-curated/github/c5debd62/skills/` and `/Users/federico.dambrosio@igenius.ai/.codex/plugins/cache/openai-curated/superpowers/c5debd62/skills/`.
- 2026-05-05 [TOOL] Verified copied skill count increased to 28 `SKILL.md` files.
- 2026-05-05 [CODE] Created `skills/deploying-domyn-swarm/SKILL.md`, `skills/deploying-domyn-swarm/references/domyn-swarm-reference.md`, and generated `skills/deploying-domyn-swarm/agents/openai.yaml`.
- 2026-05-05 [TOOL] Validated missing `name` in a Domyn Swarm YAML produces a Pydantic `ValidationError`; a config with `name`, `image`, and Slurm endpoint fields validates and builds a plan.

## Now
- 2026-05-05 [TOOL] Task complete pending user review.

## Next
- 2026-05-05 [ASSUMPTION] UNCONFIRMED: Refactor copied upstream scripts over 300 LOC only if the user wants vendored code changed rather than mirrored.

## Open Questions
- 2026-05-05 [TOOL] None blocking.

## Working set
- 2026-05-05 [TOOL] `CONTINUITY.md`
- 2026-05-05 [TOOL] `skills/`
- 2026-05-05 [TOOL] `/Users/federico.dambrosio@igenius.ai/.codex/skills/`

## Receipts
- 2026-05-05 [TOOL] `find skills -maxdepth 3 -type f` showed existing skill files for `karpathy-guidelines` and `write-lean-code`.
- 2026-05-05 [TOOL] `find ~/.codex/skills -maxdepth 3 -type f` showed installed `.system`, `codex-primary-runtime`, `karpathy-guidelines`, `rust-programming`, and `write-lean-code` skills.
- 2026-05-05 [TOOL] `git status --short` failed because the destination is not a git repository.
- 2026-05-05 [TOOL] `comm -23 /tmp/installed-skills-files.txt /tmp/repo-skills-files.txt` returned no missing files.
- 2026-05-05 [TOOL] `wc -l` counted 64 installed files and 65 repo skill files, excluding `.DS_Store`; the extra repo file is pre-existing metadata not in the installed source tree.
- 2026-05-05 [TOOL] `find ~/.codex/plugins/cache -path '*/skills/*/SKILL.md'` found 18 plugin skills: 4 GitHub and 14 Superpowers.
- 2026-05-05 [TOOL] Sequential `comm -23` checks found no missing files from local installed skills, GitHub plugin skills, or Superpowers plugin skills.
- 2026-05-05 [TOOL] `wc -l` counted 64 local installed files, 21 GitHub plugin files, 60 Superpowers plugin files, and 146 repo skill files, excluding `.DS_Store`.
- 2026-05-05 [TOOL] LOC check found copied upstream code files over 300 lines, including `skills/.system/imagegen/scripts/image_gen.py` (995), `skills/codex-primary-runtime/slides/templates/build_pro_deck_template.js` (616), `skills/codex-primary-runtime/slides/scripts/pro_deck_quality_check.js` (589), and `skills/gh-fix-ci/scripts/inspect_pr_checks.py` (509).
- 2026-05-05 [TOOL] Read Domyn Swarm README, CLI help, config models, examples, and settings/defaults code to derive skill content.
- 2026-05-05 [TOOL] `domyn-swarm --help` showed root callback may initialize/migrate the state DB even for help output.
- 2026-05-05 [TOOL] Skill validation script confirmed frontmatter, name format, description prefix, reference link, and `agents/openai.yaml` parse successfully.
- 2026-05-05 [TOOL] `wc -w skills/deploying-domyn-swarm/SKILL.md` counted 406 words; no code files were added in the new skill.
