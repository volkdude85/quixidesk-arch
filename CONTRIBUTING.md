# Contributing

## Eric Hartford has direct commit access

This package exists because QuixiDesk is good software that does nothing on Linux. Eric
Hartford wrote the thing. If he disagrees with any decision in the Linux platform layer —
an architecture call, a dependency, the way a capability degrades — he can change it
himself. He does not need to open a pull request against his own project's packaging, and
he does not need anyone's approval.

That is the point of publishing it this way rather than keeping it local.

If you are Eric and you want something reverted wholesale, revert it. The only ask is that
if it removes a measurement-backed decision, the measurement goes in the commit message so
the next person knows what was traded away.

## For everyone else

1. `git clone` and `makepkg -si`. If it does not build, that is a bug — open an issue with
   the full output.
2. Changes to `0001-linux-support.patch` should be regenerated against a pristine checkout
   of the upstream tag, not hand-edited. The patch must apply cleanly to
   `https://github.com/QuixiAI/QuixiDesk/archive/refs/tags/v<pkgver>.tar.gz`.
3. Keep the test suites passing and add a case for anything you fix. A fix without a test
   is a fix that will regress.
4. New external tools need three things: an entry in `optdepends` naming the capability it
   unlocks, an honest `capabilities()` probe, and a fall-through when it fails at runtime.

## House rules that came from real failures

* **Never claim a capability you cannot deliver.** If a backend is missing, return an error
  that names it. Silent no-ops are why the Linux side looked "finished" while doing nothing.
* **Never let one failing backend abort a chain.** Try each in turn; report all attempts.
* **Detection and execution must agree.** If the probe finds a tool, the spawn must run the
  same file.
* **A headless session is a real answer.** Do not report `x11` with no `DISPLAY`.
* **Numbers in the docs come from a benchmark.** If you change a hot path, re-measure it.

## Reporting

Upstream QuixiDesk bugs belong in the [QuixiDesk repository](https://github.com/QuixiAI/QuixiDesk).
Packaging and Linux-platform bugs belong here.
