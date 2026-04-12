# lm15 spec

Canonical type definitions, wire format, and shared test fixtures for lm15 implementations.

This repo is the **source of truth** for the lm15 type system. Language-specific implementations ([Python](https://github.com/lm15-dev/lm15-python), [TypeScript](https://github.com/lm15-dev/lm15-ts)) must conform to these definitions.

## Contents

| Path | What |
|---|---|
| [`types.md`](types.md) | Canonical type definitions — every struct, enum, and constraint |
| [`fixtures/`](fixtures/) | JSON test fixtures — request/response pairs for validation |
| [`CHANGELOG.md`](CHANGELOG.md) | Breaking changes to the spec |

## Versioning

The spec uses calendar versioning (`YYYY.MM`). Language implementations declare which spec version they target.

## License

MIT
