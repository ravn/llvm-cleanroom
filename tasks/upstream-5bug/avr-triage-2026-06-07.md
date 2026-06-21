# AVR triage of the upstream-bug queue — 2026-06-07

User directive: triage all possibly-upstream bugs against an in-tree target
before (further) upstream filing.  Toolchain: PRISTINE upstream llvm-project
`de59f9ed` at `~/llvm-upstream`, rebuilt with `LLVM_TARGETS_TO_BUILD=
"X86;AArch64;AVR;MSP430"` (the z80 fork is NOT involved except where fork
behavior is itself the subject).  AVR = the only in-tree 8-bit target
(8-bit registers, 16-bit int — same promotion pain profile as Z80).

| Bug | AVR result | Upstream-filing verdict |
|---|---|---|
| 2 TruncInstCombine Argument-leaf (FILED llvm/llvm-project#202112) | **K&R rotl 20 instructions vs ANSI 3 (6.7x) at -Os/-O2/-O3** — ANSI compiles to the canonical `lsl r24; adc r24,r1; ret`, K&R does the full 16-bit dance | **STRENGTHENED.** In-tree victim confirmed. Add the AVR datum as a comment on #202112 (user go-ahead needed). |
| 3 SimplifyCFG foldTwoEntryPHINode no-PGO | select vs branch = 6 = 6 instructions (backend equalizes). ALSO: grep shows **NO in-tree target overrides getPredictableBranchThreshold** — the isZero() path has zero in-tree constituency | **WEAKENED to fork-relevant.** Recommend NOT filing upstream; Z80 cost (−16 B) is real but stems from our expensive select lowering. Keep as fork knowledge + cl::opt demo. |
| 4 TruncInstCombine outside-user bail (staged ravn/llvm-z80#219, HELD) | sound-(a) micro-shape: escaping-cmp 10 = hand-narrowed 10 (no cost — AVR splits 16-bit ops into byte pairs natively) | **WEAKENED pending evidence.** Micro-shapes equalize on AVR; the gf_log-scale claim is untested there AND our Z80 numbers are contaminated by the soundness bug until the gate fix + re-measure. Hold #219. |
| 5 InstCombine memcpy->illegal-int fold | fold fires on 16-bit DL (i64 load/store materialized); AVR llc: folded 37 = unfolded 37 (backend swallows the illegal i64 as byte traffic) | **WEAKENED on cost, consistency argument stands** (InstCombine's own shouldChangeType gates on DL.isLegalInteger; the fold doesn't). Z80 pain was real (cpnos #87). Decide after bug-4/2 outcomes; if filed, lead with consistency, not cost. |

## Reading

The asymmetry between bug 2 (6.7x) and bugs 3/4/5 (free) on AVR: a mature
byte-oriented backend recovers raw WIDTH costs during legalization, but it
cannot recover missed IDIOMS — the K&R rotate stays unrecognized at i16, so
the cost survives.  Conversely, several of our "upstream" pain points are
partly measurements of our own immature Z80 expansion, not of the mid-end
limitation.  Bug-by-bug in-tree evidence is now the bar before any filing
(memory rule [[thorough-tests-for-upstream-bugs]] + this triage as template).

## Caveats

- Micro-shapes only; large real-code shapes (gf_log) not yet built for AVR.
- gf_log itself is under soundness suspicion (fork icmp-narrowing miscompile,
  see test_220/221/222 + the lit matrix) — its numbers are unusable until the
  gate fix + re-measure.
- MSP430 also built but not exercised (int=16=register width there, so the
  promotion-pain argument doesn't apply; kept for future comparisons).

## Addendum 2026-06-21 — bug 5 cost-witness hunt: four new shapes, all negative

Re-opened the bug-5 cost question to look for ANY in-tree AVR shape where the
memcpy->i64 fold demonstrably costs bytes/cycles (the original triage measured
only the plainest 8-byte case).  Tooling: project build `opt`+`llc` on llvm-
project-fix (AVR target present); IR hand-written, no clang in the loop.  Four
shapes measured, all confirm the 2026-06-07 verdict — no in-tree cost witness
exists.

- **Shape A (wider folds i128/i256, sizes 16/32):** MOOT.  Current upstream
  `SimplifyAnyMemTransfer` bails at `if (Size > 8 || (Size&(Size-1))) return
  nullptr` (InstCombineCalls.cpp:165) — the fold caps at 8 bytes / i64.  The
  case 16:/case 32: arms the brief described do not exist on `de59f9ed`; there
  is no i128/i256 fold to measure.
- **Shape B (memcpy + consuming compare):** fold WINS.  Folded (load i64 +
  `icmp eq i64` against folded constant + store) = 36 instr inline; baseline
  (memcpy intrinsic + `call memcmp`) = 38 instr WITH a libcall.  The i64 lets
  InstCombine inline the comparison.  Not a witness.
- **Shape C (8-byte copy in a loop):** fold marginally WINS past N=1.  Folded
  forces 4 extra callee-saved regs (8 push/pop vs 4) but the loop body is
  ~7-9 instr shorter/iteration (no pointer-juggling movw pair).  Break-even
  ~N=1.  Not a witness.
- **Shape D (i64 live across a call):** INVALID — retracted.  Initially looked
  like a witness (folded 41 instr / 10 push vs baseline 38 / 4 push, +24 cyc
  of save/restore).  But the "folded" IR I measured was hand-written as
  load->call->store, a form the fold CANNOT produce: `SimplifyAnyMemTransfer`
  always emits ADJACENT load/store (verified: `opt -passes=instcombine` on
  real `memcpy`+call IR leaves the store before the call, so the i64 is never
  live across it).  The apparent cost came entirely from my forcing the load
  before and the store after the call.  Measured the REALISTIC shapes instead:
  `copy_then_call` folded 20 instr / 0 push vs baseline 28 / 0; `call_then_copy`
  folded 30 / 4 push vs baseline 38 / 4.  Fold wins both, zero extra pressure.

**Why D matters beyond "oops":** had it gone into the filing as a cost datum,
the first reviewer would have written the real memcpy, seen it doesn't span the
call, and dismantled the claim — discrediting the genuine consistency argument
next to it (the #202 wrong-cause failure mode).  A confounded witness in a
filing is worse than none.

**Decision (2026-06-21):** bug 5 cost angle is conclusively dead on in-tree
targets — a mature byte backend always recovers the illegal-width fold during
legalization, so no shape will ever witness it.  `draft-bug5-v2.md` already
states this correctly ("no in-tree target loses real codegen quality"); no edit
needed.  Bug 5 PARKED as consistency-only with known wontfix risk (see
STATUS.md row 5).  The genuine Z80 LDIR pain that motivated this is a BACKEND
pattern-match concern, not the upstream consistency bug — filed separately as
`~/z80/tasks/z80-memcpy-ldir-pattern-match-2026-06-21.md` (Z80 project).
