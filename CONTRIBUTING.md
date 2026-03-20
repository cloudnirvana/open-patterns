# Contributing to Open Patterns

Thank you for contributing. This catalog is built by practitioners, for practitioners.

## How to Contribute

1. **Fork this repo**
2. **Create your pattern** using the [pattern template](PATTERN-TEMPLATE.md)
3. **Place it** in the appropriate category directory under `patterns/`
4. **Open a pull request** with a brief description of the pattern and where it was tested

## Pattern Template

Every pattern follows the GoF-inspired template in [PATTERN-TEMPLATE.md](PATTERN-TEMPLATE.md). Key sections:

- **Pattern in 60 Seconds** — The concept in plain language (mandatory)
- **Classification** — Category, difficulty, alternative names
- **Motivation** — A concrete scenario, not abstract theory
- **Applicability** — When to use AND when NOT to use
- **Structure** — Mermaid diagrams
- **Participants** — Components, roles, examples
- **How It Works** — Step-by-step with code/config examples
- **Consequences** — Benefits, liabilities, What Broke in Practice (mandatory)
- **Security Implications** — Attack surface, data sensitivity, failure modes, mitigations (mandatory)
- **Implementation Notes** — Variations, common pitfalls
- **Known Uses** — Real implementations
- **Related Patterns** — Complements, conflicts, alternatives

## Review Process

1. **Format check:** Does it follow the template? Is "What Broke" filled in? Is "Security Implications" filled in?
2. **Technical review:** Is the pattern sound? Generalizable? Vendor-neutral?
3. **Community validation:** Has this been pressure-tested?
4. **Merge:** Published to the catalog

## Quality Bar

**Required:**
- Real implementation (not theoretical)
- "What Broke in Practice" section filled in (no hero narratives)
- "Security Implications" section filled in
- Vendor-neutral (reference tools, don't sell them)
- Generalizable (not only applicable to one specific situation)
- Honest about tradeoffs

**Not accepted:**
- Product tutorials or vendor guides
- Patterns with no failure modes documented
- Theoretical patterns never tested in production
- Marketing disguised as engineering

## Diagrams

Use [Mermaid](https://mermaid.js.org/) for all diagrams. It renders natively on GitHub and is version-controllable.

## License

By contributing, you agree that your contribution is licensed under [CC BY 4.0](LICENSE). You retain credit as the contributor.
