# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

vlibm (`cerastes-verifier/vlibm`) is a math library (`<math.h>` / `<cmath>`) for C and C++ whose functions carry
cerastes contracts that prove them sound. It is the math counterpart of vlibc (`cerastes-verifier/vlibc`, checked out
at `../vlibc`). Both are verified with cerastes, a deductive verifier for C and C++, checked out at `../cerastes`.

Status: only the license files exist so far. There is no build system, test suite or CI yet. Add their commands here
when they land.

## Requirements for every function

- **Languages:** each function compiles and behaves the same as C23 and later, and as C++20 and later. Shared headers
  are written in the common subset of the two.
- **Platforms:** each function works on macOS and on Linux. Don't rely on declarations specific to the Apple SDK or to
  glibc unless the other platform has its own path.
- **License:** every file is dual-licensed under BSL-1.0 (`LICENSE-BSL`) and Apache-2.0 WITH LLVM-exception
  (`LICENSE-APACHE`), and its header comment carries
  `SPDX-License-Identifier: BSL-1.0 OR Apache-2.0 WITH LLVM-exception`.
- **Contracts:** each function has a cerastes contract that states what it computes, not only that it is safe to call.

## Cerastes annotations

- Contracts use the `CERASTES_*` macros from `<cerastes.h>` (source: `../cerastes/include/cerastes.h`). That header
  must be on the include path; don't copy it into this repo. vlibc follows the same rule (cerastes decision 0030
  item 1).
- With an ordinary compiler the macros expand to nothing, so annotated code is plain C or C++. The exceptions are
  `CERASTES_PURE` and `CERASTES_PREDICATE`, which expand to `inline`. Under the cerastes front end each macro becomes a
  `[[cerastes::…]]` attribute.
- The attribute namespace is closed. It holds `requires`, `ensures`, `invariant`, `decreases`, `check`, `fold`,
  `unfold`, `predicate` and `pure`. Never write `assert` (use `check`). There is no `assume`.
- Write `CERASTES_PURE` and `CERASTES_PREDICATE` first in a declaration, before `static`, `constexpr` and the return
  type.
- Placement rules are in `../cerastes/internal/decisions/0006-annotation-language-v0.md`. The user docs are in
  `../cerastes/doc/modules/ROOT/pages/annotations.adoc`.

Cerastes does not support floating point yet: `../cerastes/plan.md` lists it under "Deferred beyond M4". vlibc keeps
no floating-point contracts until that changes. `<cerastes.h>` has no vocabulary for real numbers yet: no exact real
type, `is_finite`, `real_of`, ghost code, lemmas or trusted axioms. Don't invent attributes or spec functions. If a
contract needs one, raise it as a design question for cerastes.

## Prior art

- **`../cerastes_sqrt`** is a prototype of a verified square root, ported from VeriFast's `sqrt_exact`. It shows a
  layout that serves both languages from a single proof:
  - A C ABI header declares the `f`, plain and `l` variants under one shared contract macro.
  - A generic `template <std::floating_point T>` implementation in a `.hpp` is verified once, for a symbolic `T`.
  - An `extern "C"` wrapper file is compiled with `-fno-exceptions -fno-rtti`, so C programs can link it without the
    C++ runtime.
  - `make check` type-checks the annotations with GCC or Clang, using stub builtins (`builtin_stubs.h`,
    `typecheck_annotations.py`) when the verifier cannot run.

  The prototype is older than `<cerastes.h>`, so take its structure but not its spelling. It writes attributes as
  `[[verifier::…]]` and detects the verifier with `__CERASTES__`. It also uses annotations that cerastes has not
  adopted (`ghost`, `lemma`, `trusted`, `terminates`) and a ghost theory of real numbers (`real.h`).
- **vlibc's design** is cerastes decision 0030 (`../cerastes/internal/decisions/0030-vlibc.md`). It is the closest
  model for vlibm. It covers:
  - explicit `<vlibc/…>` headers, plus shims that replace the system headers in verify mode;
  - the split between trusted contracts and verified implementations;
  - the trust rules for library directories;
  - a CI rule that compiles every header with Clang and GCC as C23 and as C++20, with `-Wall -Wextra -Werror`.

  Check with the owner before vlibm departs from that design.

## Git

- Commit to local `main` without asking, at the end of a session and after each change the owner asks for. Never
  push; the owner pushes.
- Never add a `Co-Authored-By: Claude` trailer, or any other Claude attribution, to a commit.
