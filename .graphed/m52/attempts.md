# m52 graphed-corpus implementer — iteration log

Target: C6's corpus half (`analyses/systematics.py`), byte-identical in all THREE mirrors (§7-B):
`graphed-corpus/src/`, `graphed/tests/_corpus/`, `graphed-histogram/tests/_corpus/`.
Frozen suite: `graphed tests/frozen/corpus/m52` (6.1–6.3). `git diff m52-freeze -- tests/frozen/`
MUST stay empty.

## Iteration 0 — baseline
`graphed tests/frozen/corpus` at the C1–C4-landed / C5-landed state: 3 failed (m52) / 60 passed
(m05), failing on the absent `systematics.ttbar_joint_reference` — the decomposition §5 corpus row.

## Iteration 1 — C6 corpus half

Added, per §3.4's freeze spellings:

* `btag_sf_rel_uncertainty(pt) = 0.01 + 0.05 * np.minimum(pt / 100.0, 1.0)`.
* `ttbar_joint_reference(events, *, region, jes, btag, pt_dependent=True, freeze_selection=False)`
  — `ttbar_region`'s selection/observable/binning at an arbitrary `(jes, btag)` coordinate PAIR,
  contents unrounded. `freeze_selection` takes the jet mask and event selection from nominal
  kinematics while the observable and the SF still read the shifted pT; `pt_dependent=False`
  substitutes the flat 3% of the existing `_btag_weight` rule.
* Refactor, not addition: the region cut moved out of `ttbar_region` into `_region_mask`, which the
  joint reference reuses — no second copy of the 4j1b/4j2b branch.
* Direction is a signed multiplier (`_BTAG_DIRECTION`, nominal = `0.0`), not a three-way branch, so
  the nominal leg is `sf * 1.0` — bit-identical to `ak.prod(sf)` and to the graphed side's nominal.

No catalog row: `TTBAR_FIXTURES` / `TTGAMMA_FIXTURES` and the 23 stored goldens are untouched.
No `graphed` import (G4). Not exported from `__init__.py` — the frozen fixture reaches both symbols
as module attributes.

### Gates

| gate | result |
|---|---|
| `graphed tests/frozen/corpus` | 63 passed (m52's 3 + m05's 60) |
| `graphed tests/frozen/**` (`scripts/run-tests.sh`) | all suites pass; combined coverage gate ≥90 green |
| `graphed-histogram` full suite (its mirror changed) | 206 passed |
| `graphed-corpus` own suite | 60 passed |
| three-way `diff -rq -x __pycache__` | empty |
| `git diff m52-freeze -- tests/frozen/` | empty |
| ruff check / ruff format --check (systematics.py) | clean (repo-wide `format --check` fails only on `README.md`, identically at baseline) |
| `mypy --strict` (graphed-corpus) | no issues, 6 source files |
| branch coverage of `systematics.py` from the FROZEN corpus suite | 71/71 stmts, 14/14 branches — 100% |
| determinism | probe of all three legs byte-identical across `PYTHONHASHSEED=1` / `987654` |

### Instrument margins (regenerate: `scripts/../probe_margin` recipe in the frozen test itself)

`A(graphed, pT-binned, live) = 3.343e-3`, `B(flat, live) = 8.797e-4`, `C(flat, frozen) = 1.218e-16`.
`A/B = 3.80` against the frozen `K = 3.0`; `C` is float slack on a difference of two identical sums.

### Non-vacuity of the new surface (mutants run against the FROZEN 6.1, none survived)

| mutant | killed by |
|---|---|
| `freeze_selection` ignored | control leg C reads `8.797e-4`, not machine zero |
| `pt_dependent` ignored | control leg C reads `4.656e-3` |
| `jes` ignored (silent nominal) | joint universe no longer reproduces the reference |
| `_BTAG_DIRECTION` signs flipped | joint universe no longer reproduces the reference |
