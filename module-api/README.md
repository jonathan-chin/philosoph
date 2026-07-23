# @philosoph/module-api

The contract between the Philosoph classroom-game engine and a **question module** — everything a
module author needs, and nothing about the game itself. Depends on nothing.

A question module is a self-contained plugin: it *generates* questions, *grades* them, and says what
to *reveal*. The engine knows about none of it — it is handed a registry and only ever calls the
module's methods, so a module can define correctness however it likes without any change to the
core.

## Install

```bash
npm install @philosoph/module-api
```

## The whole contract

```ts
import type { QuestionModule } from "@philosoph/module-api";

export const myModule: QuestionModule = {
  id,           // permanent, unique — it keys every recorded answer; never change it
  title,        // shown in the educator's module picker
  shortTitle,   // compact label for analytics/report charts
  description,
  generate(rng) { /* → { public: QuestionInstance, key: AnswerKey } */ },
  grade(key, submission) { /* boolean — correctness is yours */ },
  reveal(key) { /* what clients highlight, e.g. { correctOptionId } */ },
};
```

Compose a set of modules into a registry:

```ts
import { createRegistry } from "@philosoph/module-api";
const registry = createRegistry([myModule]); // throws on duplicate ids
```

## The rules that matter

1. **All randomness comes from the injected `rng`.** `Math.random()`, `Date.now()`, unsorted
   `Object.keys`, etc. break reproducibility — a session replays from its seed, so the same seed must
   always yield the same questions and the same option order. Use `rng.int`, `rng.pick`,
   `rng.shuffle`, `rng.id`.
2. **Never a second correct answer.** Build distractors that are *wrong*, and check it.
3. **Ids are forever.** A module `id` and its option ids appear in saved CSVs and manifests; changing
   one orphans every answer already recorded against it.
4. **Multiple-choice, for now.** `answerFormat` is `"multiple-choice"`; a browser client can only
   collect an interaction it has a widget for. Grading and reveal are still yours — the engine never
   inspects a key.
5. **Skills are a shared, flat vocabulary.** A question carries zero or more skill strings; reports
   group by them, so reuse the same wording across modules that mean the same thing.

## A working example

A complete, commented reference module ships with the package and is compiled/type-checked with it,
so it can't rot:

```ts
import { exampleModule } from "@philosoph/module-api/example";
```

It is intentionally **not** part of the main barrel, so it never lands in a real registry unless you
import it on purpose. Copy it to start a new module.

## How a module reaches the game

The engine is content-agnostic: it composes whatever module packages are installed. A publishable
question-module package advertises itself with the keyword **`philosoph-question-modules`** in its
`package.json` and exports `MODULES: readonly QuestionModule[]`; the engine's `modules:sync` step
discovers and composes them. Your job is just to implement this contract and publish.

## License

[PolyForm Noncommercial 1.0.0](./LICENSE) — free for noncommercial use (schools, nonprofits,
research, personal projects); commercial use is not granted.
