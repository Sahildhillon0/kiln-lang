# Kiln

**A small, friendly language that compiles to WebAssembly — and writes the glue code for you.**

> Status: **pre-alpha / design phase.** Nothing here compiles yet. Star the repo
> and watch along, or open an issue if you want to help shape it.

---

## Why Kiln?

WebAssembly is fast, portable, and sandboxed. Using it is still harder than it
should be:

- You write in a language that was designed for native machines, then fight a
  toolchain to squeeze it into Wasm.
- Every value that crosses the JS ↔ Wasm boundary (strings, arrays, objects)
  needs hand-written glue, pointer juggling, and manual memory management.
- Getting a small binary means knowing about `wasm-opt`, LTO, stripping, and a
  dozen linker flags.
- The output is a `.wasm` file plus "figure out the rest yourself".

Kiln's goal is to make the *whole* path — write, build, call from JS — boring in
the best way.

```kiln
// greet.kiln
export fn greet(name: string) -> string {
    return "Hello, " + name + "!"
}

export fn sum(xs: [i32]) -> i32 {
    var total = 0
    for x in xs { total += x }
    return total
}
```

```sh
kiln build greet.kiln
# → dist/greet.wasm
# → dist/greet.js      (ES module loader)
# → dist/greet.d.ts    (TypeScript types)
```

```ts
import { greet, sum } from "./dist/greet.js";

greet("Wasm");          // "Hello, Wasm!"   — real JS strings, no pointers
sum([1, 2, 3, 4]);      // 10               — real JS arrays
```

*(Syntax shown is a sketch and will change.)*

## Goals

| Goal | What it means |
|---|---|
| **Wasm-first** | The language is designed around Wasm's model, not ported to it. |
| **Zero-glue interop** | Exported functions get typed JS/TS bindings generated automatically. Strings, arrays, and records just work. |
| **One command** | `kiln build` produces an optimized, ready-to-import package. No separate linker, optimizer, or bundler setup. |
| **Small output** | Size is a first-class metric. Hello-world should be measured in hundreds of bytes, not megabytes. |
| **Runs everywhere** | Browsers, Node/Deno/Bun, and WASI runtimes (Wasmtime, Wasmer). |
| **Helpful errors** | Error messages that explain what went wrong and how to fix it. |

### Non-goals (for now)

- Being a general-purpose systems language that also targets native.
- Compiling existing C/C++/Rust code.
- Beating hand-tuned C on every benchmark.

## How it works

```
 .kiln source
      │
      ▼
 ┌─────────────┐   lexer → parser → AST → type checker
 │  Frontend   │
 └─────────────┘
      │  typed IR
      ▼
 ┌─────────────┐   lowers to LLVM IR, target wasm32
 │  LLVM IR    │   (optimization passes: -O2 / -Oz)
 └─────────────┘
      │  .o (wasm object)
      ▼
 ┌─────────────┐   wasm-ld links runtime (allocator, strings)
 │   Linker    │
 └─────────────┘
      │  .wasm
      ▼
 ┌─────────────┐   JS loader + .d.ts from exported signatures
 │  Bindgen    │   optional wasm-opt pass for size
 └─────────────┘
      │
      ▼
   dist/  (.wasm + .js + .d.ts)
```

Kiln uses **LLVM** as its backend for mature optimization and a well-tested
WebAssembly code generator, and **wasm-ld** (from LLD) for linking.

## Roadmap

- [ ] **v0.1 — Hello, Wasm:** integers, floats, functions, `if`/`while`, exports → `.wasm` via LLVM
- [ ] **v0.2 — Interop:** generated JS loader and `.d.ts`; strings and arrays across the boundary
- [ ] **v0.3 — Memory:** runtime allocator, structs/records, automatic memory management
- [ ] **v0.4 — Tooling:** `kiln new`, `kiln build`, `kiln run` (via a WASI runtime), good diagnostics
- [ ] **v0.5 — Ecosystem:** npm packaging, imports of host functions, source maps / DWARF for debugging
- [ ] Later: WASI Preview 2 / Component Model, threads, SIMD, a playground in the browser

## Building from source

*Not available yet.* The compiler will require:

- LLVM (with the WebAssembly target) and LLD (`wasm-ld`)
- The toolchain for the compiler's implementation language (to be decided)

## Contributing

Kiln is at the very beginning, which is the best time to influence it. Ideas,
language design discussion, and prior-art pointers are all welcome — open an
issue or start a discussion.

## License

To be decided (likely MIT or Apache-2.0).
