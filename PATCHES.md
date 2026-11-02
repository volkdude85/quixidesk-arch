# The patch series

Every patch applies to a pristine upstream release tarball
(`https://github.com/QuixiAI/QuixiDesk/archive/refs/tags/v<pkgver>.tar.gz`), in the order listed
here. `PKGBUILD`'s `prepare()` applies them in this order and `ci/ci.sh` verifies that chain from a
clean extract, so the tree that ships is the tree that was tested.

The numbers are stable — they are quoted in commit messages, issues and these documents — so the
gaps at `0006`–`0008` are deliberate rather than a missing file. Nothing depends on the series being
contiguous.

| # | file | what it does |
|---|---|---|
| 0001 | `0001-linux-support.patch` | The Linux `ComputerUse` platform layer. `linux.rs` implements capture, clipboard, synthetic input, file pickers, display enumeration, window lookup, cursor position and application launching for KDE/GNOME/XFCE/wlroots/X11/headless, plus the in-process KWin `ScreenShot2` path. Also the test suites, `NOTES-linux.md`, and the `png`/`zbus` dependencies. **This is the load-bearing patch.** |
| 0002 | `0002-voice.patch` | Read-aloud and dictation in the engine instead of the webview, because WebKitGTK implements none of the Web Speech API. TTS over a Kokoro-style HTTP endpoint or a command; STT over a streaming service's `/state` endpoint or a command. Defaults are **off**. |
| 0003 | `0003-condense-fallback.patch` | Context compaction degrades to a mechanical handoff instead of aborting when the summariser fails. |
| 0004 | `0004-skills.patch` | Skill library: discover `SKILL.md` files under the configured roots, index them, expose `ListSkills`/`LoadSkill` to the model, and render a searchable, paginated prompt block. Roots are `<data_dir>/skills`, the bundled resources dir, and anything named in `QUIXI_SKILL_ROOTS`. Nothing outside those paths is scanned. |
| 0005 | `0005-headless.patch` | `quixi --headless`: the same engine with no window (`--skills`, `--search`, `--prompt-block`, `--providers`, `--capabilities`, `--json`). Makes the layer testable in CI and usable over SSH. |
| 0009 | `0009-linux-fixes-keybindings.patch` | Bugs found by running on other machines: `is_wayland()` treated an empty `WAYLAND_DISPLAY` as Wayland and silently disabled cursor/output/window queries on real X11; `YDOTOOL_SOCKET` was honoured even when the socket did not exist; plus keyboard-shortcut discovery across KDE/GNOME/XFCE and `input_path_check()` for a zero-displacement proof that the input path works. |
| 0010 | `0010-rustify-keybindings.patch` | Removes process spawns from the keybinding readers: one `gsettings list-recursively` instead of 25 `gsettings get` calls (539 ms → 14 ms, 25 bindings → 231), and XFCE through `org.xfce.Xfconf.GetAllProperties` over D-Bus with no process at all. |
| 0011 | `0011-test-portability.patch` | Tests that asserted one machine rather than the contract: a hardcoded `wayland` session, a "> 50 KB means the screenshot worked" size heuristic, and a simulated desktop that inherited the runner's session type. |
| 0012 | `0012-kde-scenario-pinning.patch` | Pins the KDE fall-through scenario to X11, because `import` is an X11 tool and the assertion only holds there. |
| 0013 | `0013-artix-love.patch` | Non-systemd support, and the test that assumed a host can move a mouse without first asking whether it can. Ships `ydotoold.openrc`. |
| 0014 | `0014-hyprland-omarchy.patch` | Hyprland and Omarchy: cursor position from the compositor's own socket, which is the one thing the Wayland path could not answer. Also fixes two detection bugs the real machine found — `HYPRLAND_INSTANCE_SIGNATURE` alone is not proof of a live session. |
| 0015 | `0015-browser-driving-skill.patch` | A shipped `browser-driving` skill: Brave/Chromium over CDP, Firefox over WebDriver BiDi, curl only as the last resort. Installed into `<data_dir>/skills/browser-driving`. |
| 0016 | `0016-test-portability-containers.patch` | The last two tests that still asserted the host: "more than ten applications installed" (a container ships none) and "the capture error names spectacle or grim" (a host with neither names the backends it does have). Found by running the suite in the Arch CI image, and neither assertion is weakened — only made to describe the contract rather than the machine. |

## Verifying the chain

```sh
ci/ci.sh .              # pristine extract, apply, build, test, package
ci/ci.sh . --no-package # stop after tests
```

A `.rej` file anywhere in the tree means the chain is broken and the built tree is not the tree that
was tested — `ci.sh` fails on that rather than continuing.
