# EVC Office Frontend

This repository is the Én Việt Office frontend, built with React 18, TypeScript, Vite, Ant Design, TanStack React Query, Zustand, Axios, and React Router.

For frontend work, load and follow:

- `.codex/skills/SKILL.md`
- `.codex/skills/HOUSE_STYLE.md` when creating or refactoring pages, features, hooks, API modules, forms, tables, routes, or shared UI

## Non-Negotiable Rules

- Match the existing feature structure: `src/app` owns routing/pages/providers, `src/features/*` owns feature UI and local logic, shared code lives in `src/components`, `src/hooks`, `src/services`, `src/types`, `src/utils`, `src/config`, and related shared folders.
- Respect ESLint boundaries: shared imports only shared; feature imports shared plus its own feature; app imports feature and shared.
- Use the `@` alias for `src` imports and keep imports sorted by the configured lint rules.
- Reuse `HttpRequest.getInstance()` and existing API/hook patterns instead of creating raw Axios calls in components.
- Keep components focused on rendering and interaction. Put API orchestration in hooks, endpoint definitions in API modules, and reusable transformations in helpers/utils.
- Preserve existing Ant Design, table, modal, message, query key, permission, and route conventions.
- Do not introduce backend/Spring Boot/accounting instructions into this frontend repo.

## Efficient Workflow

1. Read `README.md`, the nearest route/page, feature folder, API module, hooks, shared component, and types relevant to the request.
2. Identify whether the change belongs in `src/app`, `src/features/<FeatureName>`, or shared code.
3. Implement the smallest change that matches nearby code.
4. For API work, trace endpoint usage through `src/services/httpRequest.ts`, query keys, and custom query/mutation hooks.
5. Run the smallest useful verification: `npm run lint`, `npm run build`, `npm run test`, or targeted Vitest when available.
6. Report changed files, verification, assumptions, and residual risk concisely.

## Commit Rules

Before commit, follow the README:

```shell
npm run format:fix
npm run lint:fix
```

Commit format:

```text
<commit-type>(<title>): <message>
```

Example:

```shell
git commit -m "feat(project): create crud for project"
```
