# Arch Linux PKGBUILD for QuixiDesk — native Rust (tao/wry) AI desktop workspace.
# Upstream: https://github.com/QuixiAI/QuixiDesk  (MIT)
#
# Maintainer:  volkdude85
# Contributor: Eric Hartford <https://github.com/QuixiAI> — upstream author of
#              QuixiDesk, with commit access to this repository so that anything
#              he disagrees with can be changed by him directly.
#
# The Linux platform layer in 0001-linux-support.patch was written and then measured: every
# decision in crates/quixi-platform/NOTES-linux.md is backed by a benchmark, not a preference.
#
# Upstream issues belong in the QuixiDesk repository; packaging issues here.
#
# Builds from source. The upstream prebuilt Linux tarball is also usable, but a
# source build links against the distro's own webkit2gtk/gtk, which is what you
# want on Arch (rolling libs vs. a tarball pinned against Ubuntu 26.04).

pkgname=quixidesk
pkgver=3.6556.12
pkgrel=4
pkgdesc="Native Rust desktop workspace for AI — pluggable cloud and local providers, MCP tools, projects, schedules"
arch=('x86_64')
# GCC LTO must be off. makepkg appends -flto=auto to CFLAGS/LDFLAGS by default;
# ring 0.17.14 compiles ~30 perlasm objects through the cc crate, and those
# become GCC LTO IR that rustc's link line never asks the linker plugin to
# reduce. The objects are present and nm lists every symbol, yet the link dies
# on undefined ring_core_0_17_14__* — the exact failure that cost this build
# several passes. options=('!lto') stops makepkg from adding the flag at all.
options=('!lto')
url="https://github.com/QuixiAI/QuixiDesk"
license=('MIT')
# Every entry here is a DT_NEEDED of the shipped binaries, mapped to its owner.
# pango / cairo / gdk-pixbuf2 / libsoup3 / glib2 are linked DIRECTLY by QuixiDesk;
# listing only gtk3 relied on them arriving transitively, which is wrong per Arch
# packaging policy and breaks on a system where gtk3's dependency set differs.
# Verified with `namcap` and by reading DT_NEEDED off the binary, not guessed.
depends=(
  'glibc'
  'gcc-libs'
  'gtk3'
  'webkit2gtk-4.1'
  'glib2'
  'pango'
  'cairo'
  'gdk-pixbuf2'
  'libsoup3'
  'openssl'
  'hicolor-icon-theme'
  'dbus'
  'libpulse'
  'bash'
)
makedepends=(
  'cargo'
  'rust'
  'pkgconf'
  'librsvg'
)
optdepends=(
  'git: git integration for workspace features (not linked; invoked by the app when present)'
  'spectacle: screen capture on KDE Plasma'
  'grim: screen capture on wlroots (Sway, Hyprland)'
  'xfce4-screenshooter: screen capture on XFCE (its own tool; a bare XFCE ships neither imagemagick nor gnome-screenshot)'
  'cinnamon-screenshot: screen capture on Cinnamon (ships with the cinnamon package)'
  'gnome-screenshot: screen capture on GNOME when the shell D-Bus interface is unavailable'
  'imagemagick: capture fallback (import) where no desktop tool exists'
  'kscreen: display enumeration on KDE (kscreen-doctor)'
  'xorg-xrandr: display enumeration on X11'
  'wlr-randr: display enumeration on wlroots'
  'wl-clipboard: clipboard read/write on Wayland'
  'xclip: clipboard read/write on X11'
  'xsel: clipboard fallback on X11'
  'ydotool: ALL input synthesis - typing, key combos, mouse. Requires ydotoold running.'
  'kdialog: file and folder pickers on KDE'
  'zenity: file and folder pickers on GNOME and other desktops'
  'kio: application launching via kioclient6'
  'xdotool: cursor position, and X11 window operations'
  'gnome-keyring: Secret Service backend on non-KDE desktops'
  'kwallet: Secret Service backend on KDE Plasma'
)
provides=('quixidesk')
conflicts=('quixidesk-bin' 'quixidesk-git')
source=(
  "0027-journal-and-ollama-cloud.patch"
  "0026-edition-label.patch"
  "${pkgname}-${pkgver}.tar.gz::https://github.com/QuixiAI/QuixiDesk/archive/refs/tags/v${pkgver}.tar.gz"
  "ydotoold.openrc"
  "DESIGN-NOTES-linux.md"
  "0001-linux-support.patch"
  "0002-voice.patch"
  "0003-condense-fallback.patch"
  "0004-skills.patch"
  "0005-headless.patch"
  "0009-linux-fixes-keybindings.patch"
  "0010-rustify-keybindings.patch"
  "0011-test-portability.patch"
  "0012-kde-scenario-pinning.patch"
  "0013-artix-love.patch"
  "0014-hyprland-omarchy.patch"
  "0015-browser-driving-skill.patch"
  "0016-test-portability-containers.patch"
  "0017-socket-liveness.patch"
  "0018-cli-usage.patch"
  "0019-voice-over-ipc.patch"
  "0020-dictation-referenceerror.patch"
  "0021-mac-only-capabilities.patch"
  "0022-automation-bridge.patch"
  "0023-mcp-platform-adapters.patch"
  "0024-context-button-earns-its-place.patch"
  "0025-with-skill.patch"
)
sha256sums=('bd4a041fc9b87647c0ba2eb59f204f8086a33f3f5006c8c4c1ccb11b19a62bac'
            'f090805ad11ddd9a757f3f7f54eb13c32f013feb357571a658ea3fda492d4d94'
            '0d447af0ebe5a9aa447e45cffe183c077647637994eb71ec22a29a179742484e'
            '7f0469f375334b97b3c046d0384bfd68a6f9157871c8b881f3f822ee1916e57c'
            '285268b1aacedd95c4b47619dfc20a8f229ef8ba31e5c31328d479be63a336f8'
            '79758ab34409ccbdc70e080375df6d15ad627e6ac830375519c5df48fdba4531'
            'd580530714d6841162564bd82251995bf67a80a822719aa637348c54e2b726b4'
            'ed53227a2493c19b80fe398a4f62a048a8e88947b5f3502bd474dedea5e3d633'
            'ff1a73fe454f996b749c9351f0e8b0df9f4ca7271a5ee27bc543a4a4faa1d1e0'
            '10181259810a41635a48c43f3e33d2b3e363ad5b0e24e5fa044b09fd3fd95472'
            '780223f1c89e646688ac10a36c5c084bea727f46882aa8b9289dd83fd47de62a'
            '699cf11a5ac749a7563852f92c4b229da257b843dd3dcd1145be2d585896de91'
            'bc60f83eb492d1fcda4e5e26ef1998095586f088d9c8cc8ea4c34d01054d735b'
            'f2d9c341719739a046d922c5cdfd580e4eb14a8f9926caf5db7209dc6593a0ed'
            '7d38a6f1b9d6a8b2e9765b07869672f68072eef81b6d24315646cec025893976'
            '48b49483972b25daaa046373c8f287107f0c736b2521136f1129a93a1c65cb43'
            '1649012b497ace0398f426a13ff3ef431fac42ad2c17ad08479ecf59eb09d885'
            '1ca4dd4c40bb5219c8f24f241968ffc25806ed9e59143b886e1e97d103d06b98'
            'b115045fc24a96f3d9edc103b8c1ddd867d4da29a83d61b73c38a073b765baeb'
            '34aa447b71deb28ff65d819075b85636c4c27245ab98dab6bc8fc719228ef0f6'
            '61b3c4626a7be1f33890fef816abb2ed32671449acae147c450518712727da1a'
            '13b2bfc9da3499921c4b291c4e12c0580232eceb175982fe07293f3b9021ff07'
            '30728bc944866cd8d7552bb6fc6a65953ce35e9cb06d4eff34990d9bd16e8fec'
            'ceb4036eba11f8de33794fe172e3497a3ed457dc197d71e74467be180fdfade4'
            '8f86f78d6b432ae6d7b5f29722b6d890403ec7a459a9551774574a2e47400261'
            'fdec7973c11cf751413aee8ba157cd9a7b8684c111e1baa974e3b91e2ed079d0'
            'cef94bf03388b234d141b70a935129f2fea71ebe8b017c4690e04e714ce10dc8')

_libdir="/usr/lib/${pkgname}"

prepare() {
  cd "${srcdir}/QuixiDesk-${pkgver}"
  # Persistent cargo cache: a CARGO_HOME inside $srcdir is wiped by `makepkg -C`,
  # which forces a full ~300 MB registry re-download on every clean build.
  export CARGO_HOME="${CARGO_HOME:-$HOME/.cargo}"
  # `cargo fetch` deliberately does NOT run here: the patches below add `png = "0.17"` to Cargo.toml,
  # so fetching first downloads the UNPATCHED dependency set and build() then fails offline with
  # `no matching package named png found`. It runs at the END of prepare() instead.
  # Clear every file the patches CREATE. A second build in the same tree otherwise
  # fails with "already exists! Skipping patch" and aborts prepare().
  rm -f crates/quixi-platform/src/linux.rs \
            crates/quixi-platform/examples/compat_matrix.rs \
            crates/quixi-platform/examples/platform_probe.rs \
            crates/quixi-engine/src/bin/quixi.rs \
        crates/quixi-platform/NOTES-linux.md \
        crates/quixi-engine/src/voice.rs \
        crates/quixi-platform/tests/linux_computer_use.rs \
        crates/quixi-platform/tests/capture_modes.rs \
        crates/quixi-platform/tests/bug_hunt.rs \
        crates/quixi-platform/tests/stress.rs \
        crates/quixi-platform/tests/input_synthesis.rs \
        crates/quixi-platform/tests/compat.rs \
        crates/quixi-platform/tests/compat_desktops.rs \
        crates/quixi-platform/tests/launch.rs \
        crates/quixi-platform/tests/common/mod.rs \
        resources/bundled-skills-extracted/browser-driving/SKILL.md \
        resources/bundled-skills-extracted/with/SKILL.md

  # 1) Linux platform layer: the ComputerUse surface upstream leaves as silent
  #    no-ops outside macOS, plus the KWin ScreenShot2 fast path.
  patch -Np1 -i "${srcdir}/0001-linux-support.patch"

  # 2) Voice without the webview. WebKitGTK implements none of the Web Speech API, so read-aloud
  #    and dictation are dead on Linux; this talks to a speech backend from the engine instead.
  patch -Np1 -i "${srcdir}/0002-voice.patch"

  # 3) Compaction degrades to a mechanical handoff instead of aborting when the summariser fails.
  #    context.rs half only: the subagents.rs hunks are test fixtures that assume raised subagent
  #    limits this package does not use.
  patch -Np1 -i "${srcdir}/0003-condense-fallback.patch"

  # 4) Skill library: discover SKILL.md files, index them, expose ListSkills/LoadSkill.
  patch -Np1 -i "${srcdir}/0004-skills.patch"

  # 5) --headless: run the engine CLI with no display. Fuzz 3 because this tree keeps upstream's
  #    (no fuzz needed: its Cargo.toml hunk is generated against this tree)
  patch -Np1 -i "${srcdir}/0005-headless.patch"

  # 6) Fixes found by cross-desktop testing, plus keyboard-shortcut discovery:
  #    - is_wayland() treated an EMPTY WAYLAND_DISPLAY as Wayland, silently disabling
  #      cursor_position, active_output and window_under_cursor on a real X11 session
  #    - YDOTOOL_SOCKET was honoured even when the socket did not exist, so input died with an
  #      empty error on boxes using the distro ydotoold (/run/ydotoold.socket)
  #    - list_keybindings()/find_keybinding(): normalized shortcuts across KDE, GNOME and XFCE
  #    - input_path_check(): proves the input path with a zero-displacement injection
  patch -Np1 -i "${srcdir}/0009-linux-fixes-keybindings.patch"

  # 7) Less process spawning in the keybinding readers:
  #    - GNOME: one `gsettings list-recursively` instead of 25 `gsettings get` calls (539ms -> 14ms)
  #    - XFCE:  in-process org.xfce.Xfconf.GetAllProperties over D-Bus, no xfconf-query process
  patch -Np1 -i "${srcdir}/0010-rustify-keybindings.patch"

  # 8) Test portability: the suites asserted the author's machine rather than the contract —
  #    a hardcoded "wayland" session, a "> 50 KB means the screenshot worked" size heuristic, and a
  #    simulated desktop that inherited the runner's session type. They pass on any machine now.
  patch -Np1 -i "${srcdir}/0011-test-portability.patch"

  # 9) Pin the KDE fall-through scenario to X11: `import` is an X11 tool, so asserting it is KDE's
  #    next backend after a failing spectacle only holds for an X11 session.
  patch -Np1 -i "${srcdir}/0012-kde-scenario-pinning.patch"

  # Artix Love — non-systemd systems get the same support as systemd ones.
  # The suite could not pass in a container because a test asserted that a mouse move makes
  # the pointer position known without first checking this host can move a mouse at all.
  patch -Np1 -i "${srcdir}/0013-artix-love.patch"

  # Hyprland and Omarchy. Hyprland publishes the cursor position, which is the one thing
  # the Wayland path could not answer; Omarchy is Hyprland with Quickshell on top.
  patch -Np1 -i "${srcdir}/0014-hyprland-omarchy.patch"

  # Browser driving as a shipped skill: Brave over CDP, Firefox over BiDi, curl last.
  patch -Np1 -i "${srcdir}/0015-browser-driving-skill.patch"

  # Two tests that were still asserting the host: "more than ten applications installed" (a
  # container ships none) and "the capture error names spectacle or grim" (a host with neither
  # names the backends it does have). Found by running the suite in the Arch CI image.
  patch -Np1 -i "${srcdir}/0016-test-portability-containers.patch"

  # A socket file is not a running daemon: probe by connecting.
  patch -Np1 -i "${srcdir}/0017-socket-liveness.patch"

  # 18) The CLI usage text is missing nine of its own flags.
  patch -Np1 -i "${srcdir}/0018-cli-usage.patch"

  # 19) Voice over the EIPC bridge: fetch() to a custom scheme is blocked in WebKitGTK.
  patch -Np1 -i "${srcdir}/0019-voice-over-ipc.patch"

  # 20) startEngineDictation threw ReferenceError on the first line, every click.
  patch -Np1 -i "${srcdir}/0020-dictation-referenceerror.patch"

  # 21) Do not seed the AppleScript server on a platform that cannot run it.
  patch -Np1 -i "${srcdir}/0021-mac-only-capabilities.patch"

  # 22) Define window.__quixi_automation__ so the shipped /state and /action endpoints answer.
  patch -Np1 -i "${srcdir}/0022-automation-bridge.patch"

  # 23) Sweep a stale macOS-only adapter out of an existing MCP config.
  patch -Np1 -i "${srcdir}/0023-mcp-platform-adapters.patch"

  # 24) The MCP context button only when an external server exists to browse.
  patch -Np1 -i "${srcdir}/0024-context-button-earns-its-place.patch"

  # 25) With, shipped as a skill, so the desktop knows the compiler is there.
  patch -Np1 -i "${srcdir}/0025-with-skill.patch"

  # Journal: one file per session, appended in place. Input at send, reply on completion.
  patch -Np1 -i "${srcdir}/0027-journal-and-ollama-cloud.patch"

  # Edition label: which build this is, in bold parentheses in front of the name. The
  # value is a compile-time variable so the same tree can produce any edition.
  patch -Np1 -i "${srcdir}/0026-edition-label.patch"

  # Fetch AFTER the patches so the fetched set matches what the build will use.
  cargo fetch --locked --target x86_64-unknown-linux-gnu
}

build() {
  cd "${srcdir}/QuixiDesk-${pkgver}"
  # Stock build: it says which one it is, because a second edition exists.
  export QUIXI_EDITION_LABEL="(QUIXI CORE LINUX) "
  export CARGO_HOME="${CARGO_HOME:-$HOME/.cargo}"
  export CARGO_NET_OFFLINE=true
  # Keep the toolchain deterministic: gcc/bfd as the driver, and the same
  # debuginfo level makepkg uses so ring's Rust CGUs and its C objects are
  # built under matching settings. (The undefined ring_core_*_ link failure is
  # fixed by options=('!lto') above, not by these flags.)
  # Remap the build path out of the binary. Without this, panic locations and debug info
# embed the builder's home directory -- 975 strings of /home/<user>/... in QuixiDesk
# alone -- which ships their username to everyone who downloads the package.
export RUSTFLAGS="-C force-frame-pointers=yes -C debuginfo=2 -C linker=cc -C link-arg=-fuse-ld=bfd --remap-path-prefix=$HOME=/build"
  cargo build --release --workspace --locked --lib
  cargo build --release --workspace --locked --bins
  local b
  for b in QuixiDesk quixi-chrome-native-host quixi-disclaimer quixi-guest-daemon \
           quixi-mcp-builtin quixi-mcp-worker quixi-ssh-askpass quixi; do
    [ -x "target/release/${b}" ] || { echo "quixidesk: missing binary ${b}" >&2; return 1; }
  done
}

check() {
  cd "${srcdir}/QuixiDesk-${pkgver}"
  export CARGO_HOME="${CARGO_HOME:-$HOME/.cargo}"
  export CARGO_NET_OFFLINE=true
  export RUSTFLAGS="-C force-frame-pointers=yes -C debuginfo=2 -C linker=cc -C link-arg=-fuse-ld=bfd"
  # Unit tests for the Linux platform layer: keycode table, character map,
  # desktop-entry parsing, capability consistency. The input-synthesis tests are
  # deliberately NOT run here - they inject real keystrokes and clicks into
  # whatever window has focus, so they need a human-supervised throwaway terminal
  # (QUIXI_INPUT_TEST=1) and must never fire during a package build.
  cargo test --release --locked -p quixi-platform --lib
}

package() {
  # Ship the OpenRC service. NOT under /usr/share/doc: many images — including the Artix base
  # image — carry `NoExtract = usr/share/doc/*` in pacman.conf, so pacman records the file in its
  # database and then never writes it to disk. A functional file in a documentation path is
  # invisible exactly where it is most needed. Measured: the package installed with 116 files and
  # no service file, and `pacman -Qkk` reported `/usr/share/doc (No such file or directory)`.
  install -Dm644 "${srcdir}/ydotoold.openrc" \
    "${pkgdir}/usr/share/${pkgname}/ydotoold.openrc"

  cd "${srcdir}/QuixiDesk-${pkgver}"

  # --- private tree: every helper binary plus the resources dir -------------
  # Upstream resolves helpers via current_exe()'s directory and the web UI via
  # $QUIXI_RESOURCES_DIR (or ./resources beside the binary). Both are satisfied
  # by keeping the whole portable layout together under /usr/lib/quixidesk.
  install -d "${pkgdir}${_libdir}"

  local bins=(
    QuixiDesk
    quixi-chrome-native-host
    quixi-disclaimer
    quixi-guest-daemon
    quixi-mcp-builtin
    quixi-mcp-worker
    quixi-ssh-askpass
    quixi
  )
  local b
  for b in "${bins[@]}"; do
    install -Dm755 "target/release/${b}" "${pkgdir}${_libdir}/${b}"
  done

  cp -a resources "${pkgdir}${_libdir}/resources"
  install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"

  # --- launcher ------------------------------------------------------------
  install -d "${pkgdir}/usr/bin"
  cat > "${pkgdir}/usr/bin/${pkgname}" <<'EOF'
#!/bin/sh
_libdir=/usr/lib/quixidesk
export QUIXI_RESOURCES_DIR="${QUIXI_RESOURCES_DIR:-${_libdir}/resources}"
export QUIXI_MCP_WORKER="${QUIXI_MCP_WORKER:-${_libdir}/quixi-mcp-worker}"
case ":${PATH}:" in
  *":${_libdir}:"*) ;;
  *) PATH="${_libdir}:${PATH}"; export PATH ;;
esac
exec "${_libdir}/QuixiDesk" "$@"
EOF
  chmod 755 "${pkgdir}/usr/bin/${pkgname}"

  # Headless engine driver (--skills, --search, --headless). A symlink is enough: current_exe()
  # resolves it to the real path and the binary falls back to the installed resources dir.
  ln -s "${_libdir}/quixi" "${pkgdir}/usr/bin/quixi"

  # --- desktop entry + icons ----------------------------------------------
  install -d "${pkgdir}/usr/share/applications"
  cat > "${pkgdir}/usr/share/applications/${pkgname}.desktop" <<'EOF'
[Desktop Entry]
Type=Application
Name=(QUIXI CORE LINUX) QuixiDesk
GenericName=AI Desktop Workspace
Comment=Native desktop workspace for AI with cloud and local providers
Exec=quixidesk %U
Icon=quixidesk
Terminal=false
Categories=Utility;
Keywords=AI;LLM;Chat;Assistant;Agent;MCP;Ollama;llama.cpp;
StartupNotify=true
StartupWMClass=QuixiDesk
EOF

  install -Dm644 resources/quixi-logo.svg \
    "${pkgdir}/usr/share/icons/hicolor/scalable/apps/${pkgname}.svg"

  local s
  for s in 16 24 32 48 64 128 256 512; do
    install -d "${pkgdir}/usr/share/icons/hicolor/${s}x${s}/apps"
    rsvg-convert -w "${s}" -h "${s}" resources/quixi-logo.svg \
      -o "${pkgdir}/usr/share/icons/hicolor/${s}x${s}/apps/${pkgname}.png"
  done

  # --- runtime docs --------------------------------------------------------
  install -Dm644 README.md "${pkgdir}/usr/share/doc/${pkgname}/README.md"
  install -Dm644 ARCHITECTURE.md "${pkgdir}/usr/share/doc/${pkgname}/ARCHITECTURE.md" 2>/dev/null || true
  # Platform documentation: the macOS->Linux mapping, the decisions and the
  # measurements behind them, and the dependency table. Ships inside the
  # package so it is readable without the source tree.
  install -Dm644 crates/quixi-platform/NOTES-linux.md \
    "${pkgdir}/usr/share/doc/${pkgname}/NOTES-linux.md"
  install -Dm644 "${srcdir}/DESIGN-NOTES-linux.md" \
    "${pkgdir}/usr/share/doc/${pkgname}/DESIGN-NOTES-linux.md"
}
