# Changelog

All notable changes to fab-package-nv are recorded here. The format is
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this
package follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
with the pre-1.0 rule that a breaking change bumps the MINOR number.

## [0.1.1] — 2026-11-24

### Changed

- The package is withdrawn: it continues as fab-jlcpcb-nv 0.1.0, which carries the same modules under the names `fj*` and the fixes of a first order on JLCPCB's form. No code changed.

## [0.1.0] — 2026-11-23

### Added

- `fppack.prepare`, the JLCPCB order package of a revision that passed its gates: the Gerber and drill zip, the bill of materials, the placement file, the list of corrections, the order sheet and a manifest with hashes.
- `fpcpl`, the placement file with the rotation and position corrections of the Fabrication Toolkit, and `fpdata`, the Toolkit's rotation table and the manufacturer's limits as data with their sources.
- `fpcheck`, fifteen checks of the design against the manufacturer's published limits and against itself; `fpbom`, `fpboard`, `fpsheet` and `fpzip`.

- A via is held to the page's drill and diameter limits (0.15 mm hole, 0.25 mm across, 0.1 mm over its drill); the annular ring limit is for plated pads, as the page states it.

### Known

- Nothing was uploaded to the manufacturer.
- Whether the plated and non-plated holes may share one drill file was not read on a manufacturer page.
