# WAIVER — read this before you install

QuixiDesk's Linux platform layer is the whole point of this package, and it is also the thing that
needs a warning attached. This document describes what **this** package actually does, not what a
hardened build would do.

Three things are true at once:

1. The platform layer can act as you at the keyboard and mouse, and read your screen. That is the
   feature. There is no sandbox and no allowlist.
2. **The agent loop limits are upstream's numbers, unmodified.** This package does not raise the
   round count, the subagent count, the attachment ceilings, the MCP frame sizes or the shell
   timeout. Upstream sizes them for a hosted product; this edition leaves them where upstream put
   them, and if you want them raised that is a deliberate edit you make yourself.
3. Credentials go into your OS keyring, because that is how upstream ships it.

---

## Synthetic input — the biggest one

`ydotool` writes `/dev/uinput`. The layer can **move the pointer, click anywhere, type into whatever
window has focus, and read the screen**. There is no confirmation dialog and no allowlist of
applications.

An agent running this can act as you at the keyboard and mouse: send messages, click through dialogs,
type commands into a terminal, and look at whatever is on screen — including other people's windows if
you share a desktop.

**Build around it:** run it on a dedicated desktop or a VM, not on the session where you do your
banking. Keep it off shared machines. On Wayland the compositor will not stop a program that has
`/dev/uinput` access, so "it's Wayland, it's fine" is not a protection.

`ydotoold` exists so the privileged part stays small and separate. Do not run the GUI as root to make
input work — fix the udev ACL instead.

## Screen capture

The layer captures the screen through the compositor: KWin's `ScreenShot2` over D-Bus, GNOME's shell
interface, `grim` on wlroots, or an X11 tool on X11. There is **no permission prompt on the KDE path**
— it is a direct D-Bus call, which is why it costs 29 ms instead of 921 ms. If your compositor
offers no capture interface, the call fails with an error naming what is missing rather than falling
back to something worse.

## Shell commands, at upstream's limits

The engine's shell tool keeps `timeout_seconds` clamped to **3600 s**, and the agent loop keeps its
**100-round** cap. Those are upstream's numbers.

A hung command can still hold a lock, a port or a database transaction for an hour, and a hundred
rounds is a real amount of work. **Build around it:** wrap long-running work in your own timeout, and
set a spend cap at the provider rather than relying on the loop cap to bound the bill.

## Credentials live in your OS keyring

Upstream's `credentials.rs` stores the 32-byte document key as a Secret Service entry
(`keyring::Entry`, service "QuixiDesk credentials"). This package does not change that.

Two consequences worth knowing before you file a bug:

- **You need a working Secret Service provider** — gnome-keyring, KWallet, or equivalent. Without
  one, saving a provider API key fails with `Unlock your system credential store and try again`.
  Install and unlock one first.
- Some providers report success while storing nothing. If keys vanish after a restart, test the
  keyring directly (`secret-tool store` / `secret-tool lookup`) before blaming the app.

The provider document itself is AES-256-GCM encrypted; only the key lives in the keyring.

## Network egress

The `browser-driving` skill ships a real-browser driver (Brave/Chromium over CDP, Firefox over
WebDriver BiDi) and falls back to `curl`. Anything the model looks up leaves the machine through the
browser profile it drives. That is the point of the skill, and it is also the part that is easiest to
forget when you are reasoning about what stayed local.

**If you point it at a cloud model, everything above becomes an outbound transfer**: screen captures,
file contents, transcripts, tool results. The local-model path keeps it on your machine; a cloud
endpoint does not. That is a provider decision, not a setting in this package, and it is the single
biggest switch here.

## What this is not claiming

- **Not audited.** A few weeks of work, tested on three machines (57 unattended tests plus 46
  cross-desktop checks) and no audit. Testing is not the same as an audit.
- **Not hardened.** Where a choice existed between "restrictive" and "it works", it usually went to
  "it works". The honest exceptions are the ones written down in `DESIGN-NOTES-linux.md`.
- **Not supported.** MIT, no warranty, no SLA. Issues and pull requests are welcome; obligations are
  not implied.
- **Not a security boundary.** Do not run it as root. Do not give it a shell on a machine that
  matters more than the task does.

## What I would actually do

1. Dedicated VM or dedicated desktop session, nothing else on it.
2. Local model unless you have a specific reason for cloud.
3. A provider-side spend cap, set before the first run rather than after the first surprise.
4. A working, unlocked Secret Service provider installed *before* you add an API key.
5. Your own timeout around anything that shells out.
6. Snapshot the disk before the first run. That costs nothing.
