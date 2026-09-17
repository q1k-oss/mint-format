<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset=".github/logo-dark.svg">
    <img src=".github/logo.svg" alt="MINT Format" width="88">
  </picture>
</p>

<h1 align="center">@q1k-oss/mint-format</h1>

<p align="center"><strong>~45% fewer tokens than JSON</strong></p>

<p align="center">
  MINT — Minimal Inference Notation for Tokens.<br>
  YAML-style objects and Markdown-style tables. Lossless round-trip to JSON, readable with no learning curve.
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/@q1k-oss/mint-format"><img src="https://img.shields.io/npm/v/@q1k-oss/mint-format.svg" alt="npm version"></a>
  <a href="https://www.npmjs.com/package/@q1k-oss/mint-format"><img src="https://img.shields.io/npm/dm/@q1k-oss/mint-format.svg" alt="npm downloads"></a>
  <a href="https://github.com/q1k-oss/mint/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License: MIT"></a>
  <a href="https://github.com/q1k-oss/mint/blob/main/SPEC.md"><img src="https://img.shields.io/badge/spec-v1.0-brightgreen" alt="SPEC v1.0"></a>
</p>

<p align="center">
  <a href="https://mint.q1k.ai"><strong>Playground</strong></a> ·
  <a href="https://github.com/q1k-oss/mint/blob/main/SPEC.md"><strong>Spec</strong></a> ·
  <a href="https://www.npmjs.com/package/@q1k-oss/mint-format"><strong>npm</strong></a> ·
  <a href="https://github.com/q1k-oss/mint"><strong>GitHub</strong></a> ·
  <a href="https://q1k.ai/open-source"><strong>q1k-oss</strong></a>
</p>

---

## Overview

Tokens cost money, and every existing format makes you trade something away. JSON spends
roughly 40% of its characters on braces, quotes and repeated keys. YAML fixes some of that
but still repeats every key in an array of objects. TOON is compact but cryptic. CSV is
compact and has no structure at all.

MINT takes the two notations people already read fluently — YAML-style key/value blocks and
Markdown-style tables — and uses each where it is strongest. Objects nest as YAML. Arrays of
similar objects become a table, so the column names appear once instead of once per row.

```mint
workflow:
  id: wf_7x9k2m
  name: Invoice Reconciliation
  status: awaiting_review

steps:
  | id | tool            | status    | duration | output          |
  | 1  | gmail_search    | completed | 7s       | Found 23 emails |
  | 2  | document_parser | completed | 32s      | Parsed 21 docs  |
  | 3  | sheets_lookup   | completed | 16s      | Matched 18 POs  |
  | 4  | slack_notify    | pending   | -        | -               |

messages:
  | role      | content                              |
  | user      | Run invoice reconciliation           |
  | assistant | Starting reconciliation...           |
  | assistant | Found 3 discrepancies. Please review.|
```

That is about 47% smaller than the equivalent JSON, and you did not have to learn anything
to read it.

### Benchmarks

| Dataset | JSON | MINT | Savings |
| --- | --- | --- | --- |
| User records (100 rows) | 3,245 | 1,756 | **46%** |
| API responses (50 items) | 2,891 | 1,534 | **47%** |
| Workflow states (25 steps) | 1,567 | 892 | **43%** |
| Nested configs | 892 | 523 | **41%** |
| **Average** | — | — | **~45%** |

Four things do the work:

1. **Table headers once** — column names live in the header row, not in every record.
2. **No repeated braces** — rows are delimited by `|`, not wrapped in `{}`.
3. **Minimal quoting** — most values are unquoted.
4. **Tokeniser-friendly structure** — pipes tokenise cleanly in practice.

### Compared with other formats

<table>
<tr><th>JSON</th><th>MINT</th></tr>
<tr><td>

```json
{
  "users": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
  ]
}
```

</td><td>

```mint
users:
  | id | name  |
  | 1  | Alice |
  | 2  | Bob   |
```

</td></tr>
</table>

| Against | Result |
| --- | --- |
| **JSON** | ~47% smaller, equally readable |
| **YAML** | ~35% smaller for arrays of objects |
| **TOON** | ~15% larger, far more readable |

Reach for MINT when you are sending structured data to an LLM, paying per token, want a
prompt a human can still read, or want clean diffs in version control. Reach for something
else when you need maximum compression (TOON), API interoperability (JSON), or binary data
(MessagePack).

## Highlights

- **~45% fewer tokens than JSON** — measurable savings on every LLM call.
- **No learning curve** — if you can read YAML and a Markdown table, you can read MINT.
- **Lossless** — perfect round-trip back to JSON.
- **Zero dependencies** — ~5 KB minified.
- **Typed** — TypeScript encoder and decoder with full declarations.
- **CLI included** — convert files or pipe through stdin, with a `--stats` mode.

## Install

```bash
npm install @q1k-oss/mint-format
# or
pnpm add @q1k-oss/mint-format
# or
yarn add @q1k-oss/mint-format
```

The CLI needs no install:

```bash
npx @q1k-oss/mint-format-cli input.json -o output.mint
```

## Quick start

Encode JSON to MINT:

```typescript
import { encode } from '@q1k-oss/mint-format';

const mint = encode({
  users: [
    { id: 1, name: 'Alice', role: 'admin' },
    { id: 2, name: 'Bob', role: 'editor' },
  ],
});

// users:
//   | id | name  | role   |
//   | 1  | Alice | admin  |
//   | 2  | Bob   | editor |
```

And back again:

```typescript
import { decode } from '@q1k-oss/mint-format';

const data = decode(mint);
// { users: [ { id: 1, name: 'Alice', role: 'admin' }, … ] }
```

## Usage

### Objects — YAML style

```mint
user:
  id: 123
  name: Alice
  email: alice@example.com
  active: true
```

### Nested objects

```mint
config:
  database:
    host: localhost
    port: 5432
  cache:
    enabled: true
    ttl: 3600
```

### Arrays of objects — Markdown tables

This is where MINT earns its savings:

```mint
employees:
  | id | name    | department  | salary |
  | 1  | Alice   | Engineering | 95000  |
  | 2  | Bob     | Marketing   | 75000  |
  | 3  | Charlie | Sales       | 80000  |
```

### Simple arrays — inline

```mint
tags: typescript, javascript, nodejs
numbers: 1, 2, 3, 4, 5
```

### Null values

Use `-` in a table cell:

```mint
results:
  | id | name  | score |
  | 1  | Alice | 95    |
  | 2  | Bob   | -     |
```

### Compact mode

Opt in to symbol substitution for a further squeeze:

```mint
# Standard
| status    |
| completed |
| pending   |
| failed    |

# Compact (opt-in)
| st |
| ✓  |
| ⏳ |
| ✗  |
```

### CLI

```bash
# JSON to MINT
npx @q1k-oss/mint-format-cli input.json -o output.mint

# MINT to JSON
npx @q1k-oss/mint-format-cli input.mint -o output.json

# Pipe from stdin
cat data.json | npx @q1k-oss/mint-format-cli > output.mint

# Show token savings
npx @q1k-oss/mint-format-cli data.json --stats
```

| Option | Description |
| --- | --- |
| `-o, --output <file>` | Output file path |
| `-e, --encode` | Force JSON → MINT |
| `-d, --decode` | Force MINT → JSON |
| `--compact` | Enable symbol compression |
| `--stats` | Show token count and savings |
| `--indent <n>` | Indentation spaces (default: 2) |

## API reference

### `encode(value, options?): string`

```typescript
import { encode } from '@q1k-oss/mint-format';

const mint = encode(data, {
  indent: 2,        // spaces per level (default: 2)
  compact: false,   // use symbols (default: false)
  sortKeys: false,  // sort object keys (default: false)
});
```

### `decode(input, options?): unknown`

```typescript
import { decode } from '@q1k-oss/mint-format';

const data = decode(mintString, {
  strict: true,     // throw on invalid syntax (default: true)
});
```

### `validate(input): ValidationResult`

```typescript
import { validate } from '@q1k-oss/mint-format';

const result = validate(mintString);
if (!result.valid) {
  console.error(result.errors);
}
```

### `estimateTokens(data): TokenEstimate`

```typescript
import { estimateTokens } from '@q1k-oss/mint-format';

const estimate = estimateTokens(data);
console.log(`Savings: ${estimate.savingsPercent}%`);
```

### Grammar

[SPEC.md](./SPEC.md) carries the formal specification, including the ABNF grammar. The
shape of it:

```
DOCUMENT     := (STATEMENT)*
STATEMENT    := KEY_VALUE | TABLE_BLOCK
KEY_VALUE    := KEY ':' VALUE NEWLINE
TABLE        := TABLE_HEADER TABLE_ROW+
TABLE_HEADER := '|' (COLUMN '|')+ NEWLINE
TABLE_ROW    := '|' (CELL '|')+ NEWLINE
```

## Development

A pnpm workspace with two packages: `packages/mint` (the encoder and decoder, published as
`@q1k-oss/mint-format`) and `packages/cli` (published as `@q1k-oss/mint-format-cli`).

```bash
git clone https://github.com/q1k-oss/mint.git
cd mint
pnpm install

pnpm test         # run the test suite
pnpm build        # build all packages
pnpm benchmark    # run the token benchmarks
pnpm lint         # eslint
pnpm format       # prettier
```

Roadmap:

- [x] TypeScript encoder and decoder
- [x] CLI tool
- [x] Comprehensive tests
- [x] Online playground — [mint.q1k.ai](https://mint.q1k.ai)
- [ ] Python implementation
- [ ] Go implementation
- [ ] Rust implementation
- [ ] VS Code extension

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](./CONTRIBUTING.md).

1. Fork the repository and clone your fork.
2. Create a branch: `git checkout -b feat/my-change`.
3. `pnpm install`, then `pnpm test` to confirm the suite is green.
4. Anything that changes the format itself needs a matching change to [SPEC.md](./SPEC.md).
5. Commit using [Conventional Commits](https://www.conventionalcommits.org/) and open a
   pull request.

## Related projects

MINT is part of the q1k-oss family — see [q1k.ai/open-source](https://q1k.ai/open-source).

| Package | What it does |
| --- | --- |
| [`@q1k-oss/mint-format`](https://github.com/q1k-oss/mint) | Token-efficient data format for LLM prompts |
| [`@q1k-oss/context-engine`](https://github.com/q1k-oss/context-engine) | Turns conversations and files into a versioned knowledge graph |
| [`@q1k-oss/behaviour-tree-workflows`](https://github.com/q1k-oss/behaviour-tree-workflows) | Declarative behaviour trees in YAML, durable via Temporal |
| [`@q1k-oss/kiban`](https://github.com/q1k-oss/kiban) | React components on Radix primitives and Tailwind |

## License

[MIT](./LICENSE)
