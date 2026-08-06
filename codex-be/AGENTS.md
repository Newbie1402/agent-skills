# EVC Accounting Backend

For Spring Boot accounting work, load and follow:

- `.codex/skills/SKILL.md`
- `.codex/skills/HOUSE_STYLE.md` when changing application code, including
  controllers, services, DTOs, mappers, converters, validators, repositories,
  facades, and domain helpers.

## Non-Negotiable Rules

- Follow `Controller -> Service -> Repository`.
- Controllers only handle HTTP routing, boundary validation, service delegation, and response wrapping.
- Keep business rules, branching, mapping, transactions, client calls, and repository access out of controllers.
- Use `BigDecimal` for money and preserve existing accounting lifecycle semantics.
- Prefer the smallest change that matches nearby code.

## Efficient Workflow

1. Inspect only the relevant DTO, controller, service, and tests.
2. Reuse existing services, mappers, converters, validators, and clients.
3. Implement in the owning layer; do not add speculative abstractions.
4. Run the smallest useful compile or targeted test.
5. Report changed files, verification, assumptions, and residual risk concisely.
