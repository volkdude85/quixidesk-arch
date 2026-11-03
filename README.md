# QuixiDesk for Arch Linux

[![build](https://github.com/volkdude85/quixidesk-arch/actions/workflows/build.yml/badge.svg)](https://github.com/volkdude85/quixidesk-arch/actions/workflows/build.yml)
[![license: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

An [Arch Linux](https://archlinux.org) package for [QuixiDesk](https://github.com/QuixiAI/QuixiDesk) —
Eric Hartford's MIT-licensed, native Rust (tao/wry) AI desktop workspace — plus the Linux
`ComputerUse` platform layer that upstream leaves as silent no-ops on Linux.

|  |  |
|---|---|
| what each patch does | [`PATCHES.md`](PATCHES.md) |
| every runtime, optional and external dependency, and why | [`DEPENDENCIES.md`](DEPENDENCIES.md) |
| what this leaves open, and why | [`WAIVER.md`](WAIVER.md) — **read this before installing** |
| the Linux design decisions and the measurements behind them | [`DESIGN-NOTES-linux.md`](DESIGN-NOTES-linux.md) |
| Artix / OpenRC / dinit (non-systemd) | [`ARTIX.md`](ARTIX.md) |
| Hyprland and Omarchy | [`HYPRLAND.md`](HYPRLAND.md) |

**No Flatpak.** This edition is the PKGBUILD, built from source against your distro's own webkit2gtk and
GTK. If you want a Flatpak, use the **volkdude85 edition**; a Flatpak for Core has to come from QuixiAI,
the upstream author, and until he adds one this is the package.

Upstream QuixiDesk is excellent on macOS and does nothing on Linux. This repository makes
it work: screen capture, clipboard, synthetic input, file pickers, display listing, cursor
position and application launching, all wired to real Linux backends.

Where a number appears below, a benchmark produced it.

---

## What you get

| capability | implementation |
|---|---|
| screen capture (full frame, region) | `org.kde.KWin.ScreenShot2` over D-Bus, in-process — ~29 ms full desktop vs ~921 ms for `spectacle` |
| active output name | `org.kde.KWin.activeOutputName` over the same connection |
| clipboard read/write | `wl-copy`/`wl-paste` on Wayland, `xclip`/`xsel` on X11 |
| typing, keys, mouse | `ydotool` (evdev codes; numeric keycodes, not names) |
| file and folder pickers | `kdialog` → `zenity` |
| display listing | `kscreen-doctor` / `xrandr` / `wlr-randr` |
| application launching | the parsed `.desktop` `Exec` line, spawned directly in-process |
| cursor position | `xdotool` through XWayland (a documented limitation on native Wayland), or the compositor's own socket on Hyprland |
| keyboard shortcuts | KDE/GNOME/XFCE enumeration — one bulk `gsettings list-recursively` at 14 ms returning 231 bindings, `org.xfce.Xfconf.GetAllProperties` over D-Bus, no subprocess per binding |

Every entry point returns a real `Err` naming what is missing rather than pretending to
work. `capabilities()` reports what the current session can actually do, so the UI hides
what it cannot.

## Desktop coverage

Ordered by how many people actually run them, which is also the order they were tested in.

| desktop | session | capture backend | status |
|---|---|---|---|
| KDE Plasma | Wayland | ScreenShot2, in-process | **tested live** |
| GNOME | Wayland | `gnome-screenshot` | backend chain tested by simulation |
| XFCE | X11 | `import` / `xfce4-screenshooter` | **tested live** against a headless X server |
| Cinnamon | X11 (Muffin) | `cinnamon-screenshot` | backend chain tested by simulation |
| headless (SSH, container) | none | none, and it says so | **tested live** |

Also works, lower priority: Sway, Hyprland and anything else on **wlroots**, via `grim` and
`wlr-randr`. That path needs its own code because wlroots implements `wlr-screencopy` and
KWin/Mutter do not implement it — the Wayland core protocol has no screenshot or
output-management, so every compositor picks its own. It is tested by simulation, and it is
the least common of the set.

XFCE and Cinnamon both being X11 is why they are cheap: one code path covers every X11
window manager, because RandR, the X clipboard and XTEST are the same in all of them.

Backend selection is tested by building a `PATH` containing exactly the tools each desktop
ships and asserting the right backend runs. If a backend fails, the next one is tried and
the final error lists what each one said.

## Documentation

- [`STEAMDECK.md`](STEAMDECK.md) — running this on a Steam Deck: SteamOS has a read-only root and an
  A/B image scheme, so an `pacman -U` lands in the image and is erased by the next OS update. The
  container route is the correct install, and `scripts/steamdeck-distrobox.sh` does it.
- [`FEATURES.md`](FEATURES.md) — the capability table, with what stock does and what this changes.
- [`WAIVER.md`](WAIVER.md) — the limits that were raised, stated plainly.

## Build

```sh
git clone https://github.com/volkdude85/quixidesk-arch.git
cd quixidesk-arch
makepkg -si
```

`options=('!lto')` is required and deliberate: ring 0.17's perlasm objects become GCC LTO
IR, and rustc's link line never asks the plugin to reduce them, so the link fails on
`ring_core_*`. That is not a workaround for a local problem — it reproduces on a clean box.

Runtime dependencies are the `DT_NEEDED` entries of the shipped binaries plus the data and
bus packages those imply. Everything optional is an `optdepend`, annotated with the
capability it unlocks and the session type it applies to. Install what your desktop actually
uses.

## Testing

The package runs the platform unit suite during the build (`check()`), and every suite can
be run directly:

```sh
cd src/QuixiDesk-*/
cargo test --release -p quixi-platform --lib
cargo test --release -p quixi-platform --test linux_computer_use -- --test-threads=1
cargo test --release -p quixi-platform --test bug_hunt -- --test-threads=1
cargo test --release -p quixi-platform --test stress -- --test-threads=1
cargo test --release -p quixi-platform --test compat -- --test-threads=1
cargo test --release -p quixi-platform --test compat_desktops -- --test-threads=1
cargo test --release -p quixi-platform --test launch -- --test-threads=1
```

**88 tests across ten suites; 57 of them run unattended in `ci/ci.sh`**, plus a compatibility
bench that runs the same contract in three sessions (live Wayland, X11-only, headless). The
input-synthesis, bug-hunt and stress suites inject real keystrokes and pointer events and are
gated behind `QUIXI_INPUT_TEST=1` because they move the real cursor — run them on a machine
you are not using.

Anything that moves the pointer is named `t4_` on purpose: libtest sorts tests
alphabetically, and an earlier version had a mouse test steal focus from the typing test
mid-run, producing a false failure that took an hour to diagnose.

`ci/ci.sh` verifies the **patches**, not the working tree: pristine upstream tarball, apply
the published chain, build, test, package only on success. A green working tree proves
nothing about what ships. It also fails on a hunk that applies **with fuzz** — fuzz means
context was discarded and the hunk landed somewhere it does not fully describe, which is
how a patch quietly stops meaning what it says. Offsets are fine and reported; fuzz is not.

## When something does not work

Ask the binary rather than guess. `quixi --capabilities` prints what this session can
actually do, from the same probe the UI uses to decide which capabilities to hide. In a
bare SSH shell it says so, instead of pretending:

```json
{
  "session_type": "none",
  "screenshot": false,
  "screenshot_backend": "none",
  "clipboard": true,
  "input_synthesis": true,
  "notes": ["No display session, so screen capture is unavailable regardless of which tools are installed."]
}
```

That separates "this desktop has no capture interface" from "the backend exists but failed",
which are different bugs with different owners. It is the first thing to paste into a bug
report.

## What is Rust, and what is not

The layer itself is Rust — capture orchestration, clipboard dispatch, input synthesis, display
enumeration, window lookup, file pickers, region cropping, and all the tests.

**Four things shell out on purpose, and none of them are laziness:**

* **`ydotool`** writes `/dev/uinput`, which needs root or a udev ACL. A Rust client would not
  remove that requirement, only move it inside a WebKit GUI. `ydotoold` exists so the privileged
  part stays small, auditable and separate — that is the correct design, not a shortcut.
* **`kdialog` / `zenity`** — a native chooser means linking GTK/Qt, or writing an XDG
  desktop-portal client. The portal is the one conversion worth doing next, and it is the honest
  answer for a third-party app.
* **`kscreen-doctor` / `xrandr` / `wlr-randr`** — there is no common display-listing API; doing it
  in-process would mean three separate protocols for a read-only list.
* **`wl-copy` / `xclip` / `xsel`** — `wl_data_device` via `wayland-rs` is possible; `wl-copy` is
  4 KB and this path is well-trodden.

**Capture is the exception that proves the point.** Four backends are tried in order, and KDE is
called **in-process** over D-Bus (`org.kde.KWin.ScreenShot2.CaptureArea`) rather than through
`spectacle`. That was measured, not assumed:

| path | 256×256 | 1920×1080 | 4114×1234 |
|---|---|---|---|
| `ScreenShot2` (in-process, zbus) | **2.9 ms** | **22.6 ms** | **29.2 ms** |
| `spectacle` (subprocess) | ~900 ms | ~921 ms | ~921 ms |

Roughly **30× faster**, no permission prompt, and no process spawn. Where an in-process route is
actually better, it got taken.

**Same for keyboard shortcuts.** Enumerating them through `gsettings get` was 25 subprocess spawns
per call at **539 ms**. One bulk `gsettings list-recursively` is **14 ms** — 38× faster — and returns
**231 bindings instead of 25**. XFCE goes through `org.xfce.Xfconf.GetAllProperties` over zbus, with
`xfconf-query` kept only as a fallback.

`zbus` and `png` are the only crates added. Both are pure Rust with no C dependency, so the build
stays a plain `cargo build` with no new system libraries. Full reasoning and measurements:
`crates/quixi-platform/NOTES-linux.md`, generated into the source tree at build time.

## Scope: what this changes, and what it deliberately does not

The only load-bearing part is `0001-linux-support` — the platform layer. Everything after it is
either a bug the suites found or a fix to a slow path that was worth taking:

| added | why |
|---|---|
| in-process capture and bulk keybinding reads | 29 ms against ~921 ms is too big a gap to leave on the table |
| skills directory + searchable, paginated index | a tool the model can look things up in, rather than a giant prompt block |
| voice in the engine, defaults **off** | WebKitGTK implements none of the Web Speech API, so read-aloud and dictation are dead on Linux; a service endpoint or a command works on every desktop |
| headless engine driver (`quixi`) | the engine is useful with no window, and it makes the layer testable in CI |
| the test-portability fixes | three of those tests were asserting one environment, not the contract — that is a bug in the tests |
| Artix/OpenRC, Hyprland/Omarchy, browser-driving skill | the platforms and the one workflow the first pass could not cover |

**What it does not change.** The agent loop limits, subagent counts, attachment ceilings, MCP frame
sizes and shell timeouts are upstream's numbers, unmodified. Memory is upstream's single
`memory/MEMORY.md`. Credentials go to your OS keyring, as upstream ships it. If you want any of
those raised, that is a deliberate choice you make, not something this package has made for you —
`WAIVER.md` says exactly what you are and are not getting.

**None of it is precious.** If something is not wanted, delete the patch — the series is numbered
and independent enough that dropping `0004` or `0009` breaks nothing else. If a design decision is
wrong, change it directly rather than filing a note.

The one thing worth reading closely is the platform layer itself (`0001`), because that is the part
that is load-bearing and the part most likely to have missed how a Linux backend should be
structured.

## License

MIT, matching upstream. The patch is a derivative work of QuixiDesk and carries the same
license. Upstream copyright belongs to Eric Hartford; the Linux platform layer is this
repository's.

## Credits

* **QuixiDesk** — Eric Hartford ([@QuixiAI](https://github.com/QuixiAI)), MIT. Upstream author
  and a contributor here with commit access, so anything he objects to is his to change directly.
* **Linux platform layer** — written for this repository with AI assistance, reviewed and
  directed by its maintainer.
