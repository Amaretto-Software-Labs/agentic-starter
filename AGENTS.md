# Repository Guidelines

## Project Structure & Module Organization
- `apps/api` – .NET 9 Minimal API with Identity.Base prewired; organise features in `src/Features/<Domain>` with endpoints, handlers, and EF maps together.
- `apps/web` – React 19 + Tailwind 4 SPA; keep primitives in `src/components`, domain flows in `src/features`, shared hooks in `src/lib`.
- `apps/marketing` – Astro marketing site; store content in `src/content`, serverless glue in `src/pages/api`, tokens in `src/styles`.
- `packages/` – Shared validation, telemetry, contracts; consumed via workspace imports.
- `docs/` – Up-to-date runbooks for identity, forms, email, and deployment.

## Build, Test, and Development Commands
- `dotnet build apps/api/src` then `dotnet run --project apps/api/src` to compile and start the Identity.Base-enabled API.
- `dotnet test apps/api/tests` to run backend suites with MailJet/FormFeeder stubs.
- `npm install` at root restores Node workspaces.
- `npm run dev --workspace apps/web` for the React dev server; `npm run dev --workspace apps/marketing` for Astro preview.
- `npm run lint --workspace apps/web` and `npm run test --workspace apps/web` to enforce ESLint + Vitest before commits.

## Coding Style & Naming Conventions
- C#: 4-space indent, `PascalCase` types, `camelCase` locals, `Async` suffix for `Task` methods, DTOs named `<Feature>Request`/`<Feature>Response`.
- TypeScript/React: 2-space indent, `PascalCase` components, `camelCase` utilities, `kebab-case` files; centralize Tailwind tokens in `src/styles/tokens.css`.
- Astro: content collections, `kebab-case` routes, and colocated CSS modules per page.

## Testing Guidelines
- API uses xUnit + FluentAssertions; mock MailJet and FormFeeder via DI-friendly interfaces in `apps/api/tests/Integrations`.
- Web relies on Vitest for units and Playwright for E2E; place `*.spec.tsx` beside features and smoke flows under `tests/web/e2e`.
- Marketing site runs `npm run astro check` plus Playwright smoke tests for lead funnels.

## Commit & Pull Request Guidelines
- Follow Conventional Commits referencing roadmap IDs (`feat: add onboarding wizard`).
- Keep commits scoped (API vs web vs marketing) and ship generated clients or migrations with their feature.
- PRs require a summary, screenshots for UI/Astro updates, and test evidence (`dotnet test`, `npm run test`, Playwright artifacts`).
- Ensure CI is green, refresh relevant docs (identity configs, FormFeeder schemas, MailJet templates), and resolve reviewer feedback before merge.

## Integration Notes
- Identity.Base settings live in `appsettings.Development.json.example` and `.env.sample`; keep tenant IDs, signing certs, and scopes aligned.
- FormFeeder.io forms reside in `apps/marketing/forms`; register submission webhooks through `apps/api/src/Integrations/FormFeeder`.
- MailJet templates are documented in `docs/mailjet/`; surface `MAILJET_API_KEY` and `MAILJET_SECRET` via deployment manifests and local `.env` files.
