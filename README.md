# pod-kde-selkies

The `kde-selkies` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It ships the KDE Plasma nested-compositor primitive — a
full Plasma Wayland session rendered into pixelflux and streamed over
selkies/WebRTC.

## What it provides

The KDE flavor PRIMITIVE for the selkies streaming desktop: it runs a full KDE
Plasma Wayland session (`kwin_wayland` + `plasmashell`) NESTED inside pixelflux's
virtual compositor (`wayland-1`) via the `kde-selkies-session` wrapper, so the
browser stream shows Plasma. It is the KDE analogue of the `labwc` layer — the
swappable nested-compositor seam.

The session runs headless: no seat, no SDDM, no `graphical.target`. The wrapper's
poll-for-`wayland-1` IS the ordering primitive, so it works as a pod. The encoder
is left UNSET so pixelflux auto-detects per host: NVENC on the NVIDIA flavor,
VA-API on the AMD flavor, CPU x264 otherwise — so the SAME layer streams on every
GPU config.

| Property | Value |
|---|---|
| Service | `kde-selkies-session` (`%(ENV_HOME)s/.local/bin/kde-selkies-session`, priority 12, `scope: user`) |
| Requires | `pod-selkies`, `layer-kde-shell`, `pod-pipewire`, `pod-dbus` |
| Port | `3000` (selkies web UI over HTTPS) |
| Env | `SELKIES_FRAMERATE=60` |
| Script | `kde-selkies-session` — waits for pixelflux's `wayland-1`, strips `cap_sys_nice` from `kwin_wayland`, then execs kwin directly nested |

This streamed Plasma is the pixelflux virtual-framebuffer session — intentionally
distinct from any SDDM/Plasma DRM session a physical monitor shows on a real GPU
output (that is `kde-desktop`'s job).

## How to use it

Compose the candy into a selkies KDE box (e.g. `selkies-kde-desktop`). The
service starts after pixelflux has created `wayland-1`.

```bash
charly box validate
charly check run check-selkies-kde-pod
```

The candy's own `check:` steps assert the launcher is installed and executable,
that it execs `kwin_wayland --wayland-display wayland-1` (not a headless
fallback), that `kwin_wayland` has no `cap_sys_nice`, that `kwin_wayland` /
`kdotool` / `pipewire` are present, that the session reaches RUNNING, and that
`https://127.0.0.1:${HOST_PORT:3000}/` returns `200`.

## Layout

- `charly.yml` — the `kde-selkies:` candy entity plus its `skill:` entity.
- `kde-selkies-session` — the nested-session launcher wrapper.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-selkies:kde-selkies` — the KDE nested-compositor
  primitive.
- `/charly-selkies:labwc` — the other nested-compositor primitive (the labwc
  seam).
- `/charly-selkies:kde-shell` — the SDDM-free Plasma session packages this candy
  requires.
- `/charly-selkies:selkies-kde-desktop` — the flavor metalayer composing this.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
