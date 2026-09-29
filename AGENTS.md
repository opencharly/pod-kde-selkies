# AGENTS.md — pod-kde-selkies

Standalone candy repo for the `kde-selkies` candy — the KDE Plasma
nested-compositor primitive that runs a full Plasma Wayland session nested in
pixelflux and streams it over selkies/WebRTC. The candy lives in `charly.yml` at
the repo root plus its session wrapper.

Canonical files:

- `charly.yml` — the `kde-selkies:` candy entity (description, `require`, `env`,
  `service`, `plan`) and its `skill:` entity.
- `kde-selkies-session` — the nested-session launcher (waits for pixelflux's
  `wayland-1`, then execs kwin directly nested).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-Calver history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-selkies:kde-selkies` — the owning skill: the KDE nested-compositor
  primitive, the headless de-SDDM design, and the encoder auto-detection. Load
  before editing, building, deploying, or troubleshooting this candy or its
  `kde-selkies-session` wrapper.
- `/charly-selkies:selkies-core` — the shared selkies transport and the
  supervised `[program:chrome]` service that owns Chrome for both flavors.
- `/charly-selkies:kde-shell` — the SDDM-free Plasma session packages this candy
  requires.
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; tree-position nesting, services).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs and
  `charly check run <bed>`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs, service declarations).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The live R10 witness is `charly check run check-selkies-kde-pod`: it deploys the
  shipping KDE selkies image and asserts `kde-selkies-session` RUNNING,
  `https://:3000/` → 200, Chrome CDP `/json/version` → 200, plus the deploy-scope
  `wl` KWin checks. The candy's own `check:` steps assert the launcher, the direct
  nested `kwin_wayland --wayland-display wayland-1` exec, the stripped
  `cap_sys_nice`, the `kwin_wayland` / `kdotool` / `pipewire` binaries, and the
  RUNNING session.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `kde-selkies:` candy entity in `charly.yml` and the
  `kde-selkies-session` wrapper together. The `skill:` entity in the same file is
  the owning skill's source — a candy change and its skill change land together.
- The launcher MUST exec `kwin_wayland --wayland-display wayland-1` directly
  (never `startplasma-wayland`'s headless fallback), and the caps-strip of
  `cap_sys_nice` must stay — without both, kwin runs headless and the stream is a
  black frame (RCA-proven).
- The `SELKIES_FRAMERATE` env and the service priority (`12`, after pixelflux at
  `8`) are the ordering contract; keep them in step.
- The `skill:` entity is the source for `/charly-selkies:kde-selkies`; never edit
  the generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
