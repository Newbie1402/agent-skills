---
name: evc-accounting-backend
description: Skill for EVC airline accounting backend development in Spring Boot.
---

# EVC Accounting Backend Skill

This skill is loaded by AGENTS.md.

Use this skill when working in:
- Spring Boot backend
- Accounting flows
- Excel import
- Provider mapping
- Hub/client integrations
- Financial calculations
- Ticket lifecycle changes

Read `HOUSE_STYLE.md` before implementing controllers/services.

## Core Principles

- Prefer minimal and surgical changes.
- Reuse existing architecture patterns.
- Preserve accounting lifecycle semantics.
- Keep controllers thin.
- Put business logic in services/helpers.
- Avoid speculative abstractions.

## Financial Safety Rules

- Use BigDecimal for money.
- Never introduce floating point math.
- Do not duplicate financial formulas across flows.
- Preserve existing ticket status transitions.

## Repo Working Pattern

1. Read DTO -> controller -> service flow.
2. Identify exact business rule.
3. Extend smallest existing layer.
4. Implement change.

## Validation Discipline

- DTO validation in request models/controllers.
- Business validation in services.
- Reuse existing BaseError/error enums.

## Import Flow Discipline

- Keep parsing and accounting logic separated.
- Treat import rows as dirty/incomplete data.
- Preserve parity between manual flow and import flow.