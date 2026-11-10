# Local patches vs upstream

This fork carries one permanent patch that must be re-applied after every upstream realign.

## PlayerbotsJson namespace — `src/Util/JsonUtils.h` + `src/Util/JsonUtils.cpp`

**Why:** Both this module and `mod-llm-chatter` define `JsonEscape(std::string const&)` in
global scope. When both are compiled into the same `worldserver` binary the linker emits
LNK2005 / LNK1169 duplicate-symbol errors. Upstream doesn't see this because it doesn't
ship both modules together.

**What the patch does:**

- `JsonUtils.h` — wraps all declarations inside `namespace PlayerbotsJson { ... }` and adds
  `using namespace PlayerbotsJson;` at file scope so existing unqualified call sites within
  this module compile unchanged.
- `JsonUtils.cpp` — wraps all definitions inside `namespace PlayerbotsJson { ... }`.

**How to re-apply after a realign:**

1. Hard-reset local clone to `upstream/main` (discards build-time-patched working tree files —
   those are reapplied by `build_windows.ps1` on the next build).
2. Overwrite `src/Util/JsonUtils.h` and `src/Util/JsonUtils.cpp` with the namespaced versions
   (kept in the previous commit on this branch, or see `sources.txt` in the azewrath root for
   a pointer).
3. Commit and `git push --force-with-lease origin main`.

**Companion change in `mod-llm-chatter`:** `JsonEscape` is renamed to `ChatterJsonEscape`
in that fork's `deaf575` commit for the same reason.
