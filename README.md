# fab-package-nv

**Status: WITHDRAWN. This package continues as [fab-jlcpcb-nv](https://github.com/novolang/fab-jlcpcb-nv): `novo pkg add fab-jlcpcb-nv`.** It is the same package under the name of the manufacturer it is for, with the fixes of a first order on JLCPCB's form (one origin for the Gerbers and the placement file, no part left to the manufacturer's matcher, the assembly class of each part).

A board that passed its checks is turned into the files JLCPCB asks for when a board is ordered with assembly: the Gerber and drill zip, the bill of materials in JLCPCB's columns, and the placement file (CPL) with the rotation corrections the assembly machines need. The package also writes an order sheet that a person reads before ordering, and runs fifteen checks of the design against the manufacturer's published limits and against itself.

The package places no order and calls no service. It reads the revision directory that `hardware-gates-nv` wrote, and it refuses a revision whose gates did not pass or whose files changed after the gates ran.

## Install

```
novo pkg add fab-package-nv
```

The package depends on `hardware-gates-nv`, `kicad-pcb-nv`, `sexpr-nv` and `stl-nv`. It does not start KiCad itself; the revision it reads was made by `hardware-gates-nv`, which does.

## Example

```novo
use std.env
use hgcli
use hgrun
use fppack
use fpcheck

fn main() [io, fs]
    let args = env.args()            // the project root, the revision, the design's name, the quantity
    match hgcli.locate()
        Err(e) => println(e.message())
        Ok(k)  =>
            match hgrun.run(args[1], args[2], k)
                Err(e) => println(e.message())
                Ok(_)  =>
                    match fppack.prepare(args[1], args[2], args[3], str.to_int(args[4]) ?? 0)
                        Err(e) => println(e.message())
                        Ok(p)  =>
                            println("${p.dir}: ready=${p.ready}, class ${p.class}, ${list.len(p.rows)} placed")
                            for line in fpcheck.report(p.checks)
                                println(line)
```

On `tests/fixtures/ok`, a board of three resistors, one on the bottom, with two mounting holes, this writes `hardware/fab/rev-A/jlcpcb/` and prints a line for each check. The fourteen that hold start with `ok`; the one that does not is a warning:

```
warning  assembly-class  Standard assembly takes a single board of 70 by 70 mm at the least, the board is 30 by 20: it needs a panel, which this package does not make
```

## What is written

`hardware/fab/<rev>/jlcpcb/`:

| File | Content |
| --- | --- |
| `<name>-<rev>-gerbers.zip` | every file of `gerber/`, stored without compression, with a fixed date, so that the same revision gives the same bytes |
| `bom.csv` | `Comment,Designator,Footprint,JLCPCB Part #`, one line for each group of parts, the part number taken from the symbol field the policy names (default `LCSC`) |
| `cpl.csv` | `Designator,Mid X,Mid Y,Rotation,Layer`, in millimetres, the board's auxiliary origin as zero, y upward |
| `corrections.txt` | every part whose rotation or position is not the board's own, with the rule that changed it |
| `order-sheet.md` | the size, layers, assembly class and quantity, what to choose on the web form, the corrections, the checks and the open questions |
| `package-manifest.json` | the files above with their SHA-256 |

A package whose checks hold an error is written all the same, so that the person can read what is wrong, and `Package.ready` is false.

## Placement file

The rows follow the Fabrication Toolkit (https://github.com/bennymeg/Fabrication-Toolkit, Apache-2.0), the maintained public KiCad plugin for this order. A part is placed at the centre of its pads, not at its anchor, with the board's auxiliary origin subtracted and y flipped. A part on the bottom has its rotation taken from 180. The rotation table of the Toolkit, `plugins/transformations.csv` at commit `7f627b6be76081b962161b9eb9fe3a326a18c915`, is carried in `fpdata` row for row; the first row whose pattern matches a footprint's name adds its rotation and moves the part by its offset, turned with the part. A footprint field `FT Rotation Offset` or `FT Position Offset` adds to that, and `FT Origin = Anchor` places the part at its anchor. Every part that changes is listed in `corrections.txt` and the order sheet, so that a person checks it in JLCPCB's preview.

The one row of the table with a negative look-ahead, `^SOP-(?!18_)`, is written as two rows, `^SOP-18_` and `^SOP-`, which give the same answer.

## The checks

| Check | Severity | What it measures |
| --- | --- | --- |
| designators-unique | error | no designator twice on the board or in the bill |
| part-numbers | error | every placed part has a part number; a part marked do-not-populate or left out of the position file is not placed on purpose |
| part-number-format | error | a part number is `C` and digits |
| bom-cpl-agree | error | the bill and the placement file name the same parts |
| footprints-agree | warning | the bill's footprint is the board's |
| positions-agree | error | KiCad's own position file gives the anchors the placement file starts from |
| assembly-class | warning | Economic or Standard assembly, and the sizes and quantities of that class |
| board-size, layers, thickness | error | within the manufacturer's range |
| track-width | error | the narrowest track against the least for the layer count |
| clearance-setting | warning | the project's rule minimums against the manufacturer's least |
| drill | error | via, plated and non-plated drills, a via's diameter over its drill and not below 0.25 mm |
| annular-ring | error | the smallest ring of a plated pad (a via is held to the drill check) |
| rotations | warning | how many parts have a correction to review |

## Read from JLCPCB's pages

The figures were read on 23 November 2026 and are a statement of what the pages said that day.

- The columns of the bill, from the manufacturer's sample file `Sample-BOM_JLCSMT.xlsx`: `Comment`, `Designator`, `Footprint`, `JLCPCB Part #`. The sample carries `(optional)` after the last.
- The columns of the placement file, from https://jlcpcb.com/help/article/pick-place-file-for-pcb-assembly: `Designator`, `Mid X`, `Mid Y`, `Rotation`, `Layer`.
- The limits, from https://jlcpcb.com/capabilities/pcb-capabilities and https://jlcpcb.com/capabilities/pcb-assembly-capabilities, for a 1 oz board: tracks and gaps 0.10 mm on two layers and 0.09 mm on more, drills from 0.15 mm, plated-pad annular ring 0.18 mm (0.25 mm recommended) on two layers and 0.15 mm (0.20 mm recommended) on more, a via of at least 0.25 mm over a hole of 0.15 mm, a non-plated hole from 0.5 mm, thickness 0.4 to 4.5 mm, up to 32 layers, the largest board by layer count. Economic assembly: one side, 2 to 6 layers, 0.8 to 1.6 mm, 10 by 10 to 470 by 500 mm, 2 to 50 pieces. Standard assembly: both sides, 70 by 70 to 460 by 500 mm, 2 to 80000 pieces.
- The steps of the order, from https://jlcpcb.com/help/article/how-do-i-place-a-pcba-order.

## What is not verified

- Nothing was uploaded. Whether the form accepts these files as they stand, and what its matcher answers for each part number, is not tested.
- Whether JLCPCB takes the plated and non-plated holes in one drill file, as the gates' export produces, was not read on a JLCPCB page.
- The Toolkit's rotation table is the answer of a plugin that people use; a part it does not list can still face the wrong way in the preview.
- Stock, price and whether a part is an extended part are not looked up.
- Copper weight, surface finish and solder mask colour are not in a board file and are left to the form.
- A board below the minimum size of its assembly class needs a panel; the package says so and does not make one.

## Tests

```bash
novo test tests/fp_zip_tests.nv        # CRC-32 and the stored archive, read back by unzip when it is installed
novo test tests/fp_data_tests.nv       # the tables and their figures
novo test tests/fp_board_tests.nv      # the facts read from a board
novo test tests/fp_cpl_tests.nv        # the placement file and its corrections
novo test tests/fp_bom_tests.nv        # the bill read and written
novo test tests/fp_check_tests.nv      # every check, passing and failing
novo test tests/fp_sheet_tests.nv      # the order sheet and its questions
novo test tests/fp_pack_tests.nv       # a recorded revision made into a package, and every refusal
novo test tests/fp_fault_tests.nv      # messages and codes
bash tests/coverage.sh                 # line coverage over src/, merged across the suites
```

`tests/fixtures/rev_ok` is the output of the gates, run once with KiCad 10.0.8, so no test needs KiCad.

## Licence

Apache-2.0. See `LICENSE`. The rotation table is the Fabrication Toolkit's, Apache-2.0.
