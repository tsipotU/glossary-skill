# Glossary Skill for Claude Code

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that manages a project-wide ubiquitous language glossary — a shared vocabulary as version-controlled YAML files. Bridges the gap between what developers call things in code (`cart_item`), what users see in the UI ("Shopping Bag"), and what the team says in meetings ("basket").

Inspired by [Domain-Driven Design](https://martinfowler.com/bliki/UbiquitousLanguage.html) and compatible with [Contextive](https://docs.contextive.tech/community/guides/setting-up-glossaries/).

## What it does

- **Explore** — look up terms, trace relationship chains, reverse-lookup `code_name` to `ui_label`
- **Add** — conversationally add terms without writing YAML; infers what it can, asks only what it can't
- **Maintain** — health checks: orphan detection, stale terms, broken relationships, cross-context conflicts, Mermaid diagrams
- **Scaffold** — bootstrap a glossary for new projects with bounded contexts seeded from the codebase

## Install

```bash
claude plugin marketplace add tsipotU/glossary-skill
claude plugin install glossary@tsipotu-glossary
```

## Usage

The skill triggers on phrases like "glossary", "terminology", "ubiquitous language", "what does X mean", "define a term", "add a term", "glossary health", or "stale terms".

```text
> What does Order mean?
> Add Cart Item to the checkout glossary
> How does Cart relate to Order?
> Show me stale terms
> Set up a glossary for my project
```

## Glossary format

Terms live in `.glossary.yml` files organized by bounded context:

```yaml
# checkout.glossary.yml
context:
  name: Checkout
  description: The process of completing a purchase.

terms:

  # ─── Cart ────────────────────────────────────
  - name: Cart
    definition: A temporary collection of items a user intends to purchase.
    aliases:
      - Shopping Cart
      - Basket
    code_name: cart
    ui_label: "Shopping Bag"
    status: active
    tags:
      - user-facing
    related:
      - contains: Cart Item
      - leads_to: Checkout Session
        when: "Cart contains at least one item"
```

### Relationship types

| Type | Meaning | Auto-inferred inverse |
| --- | --- | --- |
| `contains` | Parent owns children | `part_of` |
| `references` | Uses but doesn't own | `referenced_by` |
| `leads_to` | Flow to a different entity | `follows` |
| `transitions_to` | Same entity, new state | `transitions_from` |
| `replaces` | Deprecated term superseded | `replaced_by` |

Inverses are auto-inferred — only store forward relationships in YAML.

## How other tools use it

The glossary is plain YAML in a known location. Any tool can read it — no special API needed. The file format is the API.

| Tool | Integration |
| --- | --- |
| **Gherkin skill** | Reads `ui_label` for steps, flags new terms not in glossary |
| **Contextive** | VS Code hover definitions (compatible format) |
| **Code review** | Flags identifiers that don't match `code_name` entries |
| **CI linting** | Validates code identifiers against glossary |

## Changelog

### v1.0.0

- Initial release
- Three modes: explore, add, maintain
- Scaffolding for new projects with bounded context proposals
- 5 typed relationships with auto-inferred inverses and `when` conditions
- Tiered loading strategy (pointer → scoped → full)
- Contextive-compatible YAML format
- Term schema reference bundled

## License

MIT
