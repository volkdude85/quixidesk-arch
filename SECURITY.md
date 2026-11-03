# Security policy

## What this software can do

Worth stating before anything else, because it defines what a vulnerability means here. The Linux
platform layer can **move the pointer, click, type into whatever window has focus, read the screen,
read and write the clipboard, and launch applications**. That is the feature. `WAIVER.md` describes
what is and is not bounded about it.

## Reporting

Open a **private security advisory** through the repository's Security tab rather than a public
issue, for anything in this category:

* the capability layer reaching a window, output or clipboard it was not asked to touch
* a backend that reports success without doing the work — a silent no-op is why the Linux side
  looked finished while doing nothing, and a new one is a defect worth knowing about
* anything that lets a captured image, transcript or clipboard payload leave the machine that was
  not on its way to the provider the user configured
* a build or install path that writes outside the package's own directories

Ordinary bugs — a desktop whose capture backend fails, a crash, a missing dependency — are fine as
public issues. The issue form asks for `quixi --capabilities` output, which is usually enough to
tell the two apart.

## What is not a vulnerability

* The agent can act as you at the keyboard and mouse. That is the documented design, not a bypass.
* Limits on the agent loop are upstream's. If you think they are too loose, that is a configuration
  decision for your own build, not a flaw in this packaging.
* Running it as root is unsafe and unsupported. So is pointing it at a machine whose input you do
  not control.

## Supported versions

The current `pkgver` in the PKGBUILD, and nothing else. This is a packaging repository: it tracks
upstream releases rather than maintaining branches, so a fix lands as a new version.

## What is not in this repository

Audited at packaging time, and it holds for every release cut from this tree:

- **No credentials.** No API keys, tokens, account identifiers or `.env` contents. The single string
  `OLLAMA_API_KEY`-style occurrence in the series is an environment *variable name* in a provider
  definition (`OLLAMA_API_KEY`), which is the name the app reads, not a value.
- **No personal data.** No home directory paths, no street address, no phone number, no personal email,
  no legal or family names. Patches are cut against `a/`…`b/` paths, not against a machine.
- **No runtime state.** `providers.json`, `credentials.key`, `preferences.json`, `voice.json` and the
  data directory are refused by `.gitignore`; a key somebody pasted lives in their data directory and
  never in a tree.
- **Nothing macOS-shaped is advertised on Linux** (0028), so a Linux build cannot be asked for a
  permission prompt or an accessibility tree that does not exist here.

Re-run the audit any time with:

```sh
git grep -nIE '/home/[a-z]|\bSean\b|sk-[A-Za-z0-9_-]{20,}|ghp_[A-Za-z0-9]{20,}|hf_[A-Za-z0-9]{20,}|\b(971|503)[-. ]?[0-9]{3}[-. ]?[0-9]{4}\b' -- .
```
