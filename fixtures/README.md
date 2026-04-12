# Test Fixtures

JSON files containing canonical serialized forms of all lm15 types.

## `canonical.json`

A flat object where each key is a fixture name and each value is the serialized form of a type instance. Implementations must:

1. **Deserialize** each fixture into the corresponding type
2. **Re-serialize** it back to JSON
3. **Assert** the output matches the original (round-trip)

Fixture naming convention: `{type}_{variant}`, e.g. `part_text`, `request_with_tools`, `stream_end`.

## Adding fixtures

1. Add the case to the generation script in the Python repo (`scripts/gen_fixtures.py`)
2. Run it to regenerate `canonical.json`
3. Update this repo
4. All language implementations' CI will catch any drift
