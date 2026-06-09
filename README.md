# ARCHIVED

**This repository is archived and must not be used as a reference.**

It was formerly the source of truth for the lm15 type system, but it
documents the v1-generation design (lm15-python 0.2.0): nested `DataSource`,
`ToolConfig`/`ReasoningConfig`, `LMRequest`/`LMResponse`, accessor-style
`Part`, `ULMError`. The current reference implementation (`lm15-python2`)
removed or renamed most of these; the fixtures in `fixtures/canonical.json`
do not round-trip through the current serde.

The single source of truth for lm15 behavior is now the
**`lm15-contract`** repository (fixture corpus + `AUTHORITY.md` precedence
rules). Type-system documentation lives with the reference implementation
until its extraction into the contract repo.

Nothing here will be updated. It is kept only for the historical record of
the v1 design.
