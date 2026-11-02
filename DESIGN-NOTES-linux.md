# Design choices made for Linux — and what we learned

Notes from porting QuixiDesk's desktop integration to Linux without Apple's frameworks. These are
measurements and decisions from daily use on KDE Wayland, GNOME Wayland and XFCE X11, not
preferences. Where we made a choice for you, it is written down here with the reasoning so you can
disagree with it on the evidence rather than guess at intent.

Nothing here changes behaviour on macOS. Every decision is behind a platform check.

---

## 1. Voice cannot come from the webview on Linux

Upstream read-aloud and dictation call the web standards — `window.speechSynthesis` /
`SpeechSynthesisUtterance` for output, `SpeechRecognition` / `webkitSpeechRecognition` for input —
and rely on the system webview to provide them. That works on macOS because WKWebView routes those
to Apple's speech stack.

**WebKitGTK implements none of it.** Measured on webkit2gtk 2.54.2, asking the actual webview:

```
speechSynthesis            undefined
SpeechSynthesisUtterance   undefined
getVoices                  absent
SpeechRecognition          undefined
webkitSpeechRecognition    undefined
getUserMedia               undefined
```

So on Linux both features are dead — and this is not a desktop-environment problem. XFCE, GNOME,
KDE and sway all use the same WebKitGTK, so a per-DE fix does not exist.

**Our choice:** implement voice in the engine instead of the webview. `crates/quixi-engine/src/voice.rs`
talks to a speech backend over HTTP or as a child process, which makes it identical on every Linux
desktop. Backends are pluggable:

- TTS — `kokoro { url, voice, speed }` (POST `{text, voice, speed}`, WAV back) or `command`
- STT — `streaming { url }` (a streaming service with a `/state` endpoint that accumulates a
  transcript), `command`, or `off`

Defaults are deliberately **off**. Pointing at a service that may not exist on your machine would
fail confusingly, and picking a voice is a taste decision that belongs to the user. Configuration is
`<data_dir>/voice.json`, written with the default shape on first run so it is discoverable.

### Two things we got wrong first, and what fixed them

**The first word of speech gets eaten.** Opening an audio sink — Bluetooth especially — loses the
first fraction of a second of whatever you play into it. Every spoken reply lost its opening word.
Fix: `with_lead_in()` injects silent frames into the WAV before playback (250 ms default), so what
falls into the sink-opening hole is silence rather than a syllable. It walks RIFF chunks instead of
assuming a 44-byte header, and handles stereo and 16/24-bit.

**The first word of dictation gets eaten too.** If you stop the old mic consumer before starting the
new one, the gap swallows the start of the sentence. The engine never stops the talk service to
start it — open the new consumer first, release the old one after.

### Current state in this package — read this before testing

The module is **present and compiles**: `crates/quixi-engine/src/voice.rs` is in the tree, registered
as `pub mod voice`, and its symbols are in `libquixi_engine.rlib`. It is **not yet wired to the UI**,
which means nothing in the running app calls it — the linker drops unused functions, so you will not
find it by poking at the binary.

What that means practically: voice does not work out of the box in this build. The remaining work is
wiring, in the app layer:

1. A speaker control on assistant messages → `voice::speak(&cfg, text, 250)`.
2. A microphone control in the composer → start the STT backend, then poll `voice::transcript(&cfg)`
   and diff against what has already been inserted.
3. A settings row → `voice::status(&cfg)` already reports honest availability for both directions, so
   the UI can grey the buttons out rather than offering something that silently does nothing.

That is a small amount of glue and it is deliberately left to you, because the shape of the UI is
your design decision and not ours to make from a machine we cannot see.

### Where this needs help

It works, but it is not as fluid as the Apple stack. Two things would improve it most:

1. **The sink-open latency.** Padding 250 ms of silence is a blunt fix. A pre-warmed sink, or a
   latency query returned by the backend, would let it be tighter and less wasteful.
2. **The backend abstraction.** It is two hand-rolled variants each. A proper trait with a factory
   would make adding Piper, Coqui, whisper.cpp or a cloud provider a small job instead of a patch.

---

## 2. Capture: pick the cheapest backend, and never trust the first one

Screenshot latency decided the design. Measured on the same machine, same frame:

| method | 256×256 | 1920×1080 | full desktop (4114×1234) |
|---|---|---|---|
| KWin ScreenShot2 (D-Bus, in-process) | **2.9 ms** | **22.6 ms** | **29.2 ms** |
| `spectacle -b -f` | ~1250 ms | ~950 ms | ~921 ms |

`spectacle` was the obvious choice and it is two orders of magnitude slower, because it writes a
file, launches a process and reads it back. KWin's `ScreenShot2` D-Bus interface hands back the
framebuffer directly over a pipe.

**Our choice:** the capture chain tries backends in cost order and **never aborts on the first
failure** — every backend is attempted and the error lists what each one said. A desktop that has
`grim` but not `spectacle` still works, and when everything fails you get an explanation instead of
a generic error.

Region capture **crops in Rust**, not with ImageMagick: invoking `import` cost 1250 ms and often
failed outright on GNOME. In-process cropping is 1.4 ms.

---

## 3. X11 goes through the protocol, not through shelling out

For X11 sessions we use `x11rb` (pure Rust, no `libX11`) rather than invoking `xrandr`,
`xdotool` and `import`. `GetImage` for capture, RandR for displays, `QueryPointer` for the cursor,
and `Xft.dpi` read from `RESOURCE_MANAGER` for scale — no subprocesses, no parsing of human-oriented
output.

This also made the depth handling explicit: a 16-bit root is RGB565 in 2 bytes per pixel, a 24/30-bit
root is BGRX in 4. Both paths are covered by tests that assert which branch ran, because "the bytes
looked wrong" is not a useful signal.

---

## 4. Where a value is genuinely unknown, return nothing rather than a number

On Wayland there is no client API for the global cursor position. XWayland will answer
`XQueryPointer` — with a stale coordinate, often the screen centre. We were reporting that as truth,
and it poisoned anything derived from it (including "which output is active").

**Our choice:** `get_cursor_position` returns `None` on Wayland instead of a plausible-looking lie,
and callers that can work without it do so. The same principle applies throughout: when a tool is
missing, the error names the tool. There are no silent no-ops left in the Linux path.

That is a deliberate trade — some callers now have to handle "unknown" — and it is the right one,
because a wrong number is worse than an honest absence.

---

## 5. `ydotool` needs a delay on Wayland, and zero on X11

Typing through `ydotool` with `-d 0` is fast and **silently corrupts text on GNOME Wayland**:
`qx-26736` arrived as `qx2673d` — dash lost, digits transposed, no error returned. With `-d 25`
it is byte-exact.

On X11 the opposite is true: `-d 0` is correct and the delay only costs time. So the delay is
chosen by session type, not fixed.

Typing speed measured at `-d 0`: 133 µs per key. The old batch size was the real bottleneck; raising
it took 41 characters/second to 7,401.

---

## 6. Session type is three values, not two

An early version answered "x11" whenever `DISPLAY` was set — including in containers, over SSH, and
in CI, where there is no X server at all. Code branched on that and failed in confusing ways.

**Our choice:** `wayland` / `x11` (only when a display actually exists) / `none`. Anything that
needs a display checks for `none` first.

---

## 7. What we did not change

- **No behaviour on macOS is altered.** Every change is behind `#[cfg(target_os = "linux")]` or a
  runtime session check.
- **No silent fallbacks were added.** Where a capability genuinely does not exist (per-app input
  permissions, TCC, save-panel focus), we return `not-applicable` / `[]` / `false` rather than
  inventing an equivalent.
- **No new heavy dependencies.** The additions are pure Rust: `zbus`, `png`, `x11rb`.

---

## 8. Measured, not assumed

If you take one thing from this: almost every decision above came from measuring something that
looked fine. `spectacle` looked fine. `-d 0` looked fine. The Wayland cursor looked fine. Each was
wrong by a factor, or silently corrupting, and none of it was visible without timing it or checking
the output byte-for-byte.

Tests included with the Linux layer assert which code path ran, not merely that the result was
non-empty — because "it returned something" is exactly how the silent failures survived.

---

*Linux layer, skills integration, headless driver and voice: written for this fork. Memory design
credit: MemPalace by Milla Jovovich and the MemPalace project (MIT).*
