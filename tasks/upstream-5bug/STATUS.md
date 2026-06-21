# 5-bug upstream queue — status 2026-06-10

Reference build: upstream llvm-project at `de59f9ed` (~/llvm-upstream on
sonnyboy, built).  Process: feedback_explain_before_filing — one bug at a
time, explicit per-filing user go-ahead; report-only (no fix patches).

Watcher: remote routine `llvm-upstream-bug-watch` (trig_012Thn7hHeabxS59DsQPzkRS,
daily 06:00 UTC) now has the watched issue list HARDCODED in its prompt (decoupled
2026-06-21 when upstream-5bug migrated here from ~/z80).  To add a new issue,
update the routine prompt at claude.ai/code/routines — no longer reads this file.
Reports at claude.ai/code/routines.

**Net state (2026-06-21):** queue closed out.  Bug 2 filed (#202112) and
watched.  Bug 1 dropped to #217.  Bug 3 moved to fork tracker #223.  Bug 4
filed (#204098) and watched.  Bug 5 PARKED 2026-06-21 — consistency-only,
cost angle conclusively dead in-tree after the four-shape hunt (see
`avr-triage-2026-06-07.md` addendum).  No bug remains AWAITING a verdict.
The Z80 LDIR backend item split off to
`~/z80/tasks/z80-memcpy-ldir-pattern-match-2026-06-21.md` (Z80 project).  AVR triage in
`avr-triage-2026-06-07.md` is the load-bearing input — every "WEAKENED",
"DROP", or "PARK" call below traces to a row in that doc.

| # | Bug | Status |
|---|-----|--------|
| 1 | deleteDeadLoop SSA malform | DROPPED from upstream queue — caller-contract violation by Z80PatternFillRecognize (was Z80LoopIdiomFill until 2026-06-09) + live HEAD regression. Filed **ravn/llvm-z80#217** (open; fix = formDedicatedExitBlocks in pass + revert generic LoopUtils divergence + clang-shaped lit test). |
| 2 | TruncInstCombine Argument-leaf | **FILED UPSTREAM: llvm/llvm-project#202112** (2026-06-07, explicit user go-ahead after staged iteration on ravn/llvm-z80#218, now closed w/ cross-ref). Two-voice form: user summary + attributed Claude deep-dive; line-exact L95-L105 permalink; rj_sb_inv provenance with corrected 147/16/31 numbers. AVR oracle STRENGTHENED (K&R rotl 20 vs ANSI 3 instr at all -O levels). Watch for maintainer responses. |
| 3 | SimplifyCFG foldTwoEntryPHINode | **MOVED BACK TO Z80 PROJECT 2026-06-21.** Fix is Z80-backend modelling (`getPredictableBranchThreshold()` override + accurate `select` cost in TTI) — belongs in `~/z80`, not the clean room.  Artefacts (`bug3-twoentry-phi-no-pgo.ll`, `draft-bug3.md`) moved to `~/z80/tasks/upstream-5bug/`.  Fork tracker: `ravn/llvm-z80#223`. |
| 4 | TruncInstCombine outside-user bail | **FILED UPSTREAM 2026-06-16: llvm/llvm-project#204098** ("[AVR][AggressiveInstCombine] TruncInstCombine bails on K&R uint8_t loop with outside-graph icmp user — emits 16-bit code where 8-bit is achievable"). AVR-targeted standalone filing with user's intro + AI disclosure, gf_log reducer (verbatim from public-domain AES-256), 70 B vs 48 B size delta, 825K vs 596K cycle witness on simavr, IR diff, 8-variant source-level workaround table, cause + direction sketch (no patch). Per `feedback_explain_before_filing` user gave explicit per-filing go-ahead.  Body at `issue-body-avr-gflog-missed-opt.md`; supporting artefacts: `avr-gflog-missed-opt.c`, `avr-gflog-runtime.c`, `gflog-variants.c`, `avr-gflog-workaround-2026-06-15.md`, `avr-gflog-oracle-2026-06-15.md`, `plan-bug4-unpark-2026-06-15.md`. Watch for maintainer responses. |
| 5 | InstCombine memcpy->illegal-int fold | **PARKED 2026-06-21** as consistency-only with known wontfix risk. Cost angle conclusively dead on in-tree targets: the 2026-06-21 four-shape hunt (see `avr-triage-2026-06-07.md` addendum) found NO AVR witness — shape A moot (fold caps at 8 B, no i128/i256 arm on de59f9ed), shapes B/C the fold WINS, shape D was a confounded hand-written form (load->call->store, which the fold never produces) and was RETRACTED. A mature byte backend always recovers the illegal-width fold during legalization. CONSISTENCY argument still stands (`shouldChangeType` gates on `DL.isLegalInteger` at `InstructionCombining.cpp:307`; `SimplifyAnyMemTransfer` at :165 doesn't) — `draft-bug5-v2.md` carries it correctly and needs no edit, but filing-as-policy-only carries real wontfix risk so it stays parked pending user priority. The genuine Z80 LDIR pain that motivated this is a BACKEND pattern-match item, split off to `../z80-memcpy-ldir-pattern-match-2026-06-21.md`. Provenance: cpnos `init.c:435` `__builtin_memcpy(&msg[DAT], login_pwd, 8)` (#73 -> #87 -> local guard `475a65378517` = InstCombineCalls.cpp:172). |

Draft bodies kept on disk:
- `draft-bug2.md` (filed as #202112)
- `draft-bug3.md` (DROPPED; kept for fork-internal context)
- `draft-bug4.md` (HOLD on #219 + icmp-narrow gate)
- `draft-bug5.md` (HOLD; original cost-led framing)
- `draft-bug5-v2.md` (NEW 2026-06-10; consistency-led reframing, AWAITING verdict)

`repro182.c` + `issue182reg.md` = the #217 filing (already filed; these fork-specific files stayed in `~/z80/tasks/upstream-5bug/` when this folder migrated).
