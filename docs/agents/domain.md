# Domain Docs

Before exploring, read:

- root `CONTEXT.md`;
- relevant ADRs under `docs/adr/`, when present.

Missing files require no warning. Domain-modeling skills create ADRs lazily.

Use glossary terminology in issues, plans, tests, and code. Avoid synonyms explicitly rejected by `CONTEXT.md`.

Surface conflicts with existing ADRs instead of silently overriding them.

## Layout

Single-context repository:

```text
/
├── CONTEXT.md
├── docs/adr/
└── src/
```
