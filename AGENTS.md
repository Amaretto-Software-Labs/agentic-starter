# Repository Guidelines

## Project Structure & Module Organization
- `backend/Identity.Host` - Identity Base host wired for OpenID Connect, MFA, and external providers; keep composition in `Program.cs` and configuration under `appsettings.*`.
- `backend/Identity.Api` - .NET 9 minimal API; feature folders live in `Features/<Domain>` with endpoints, handlers, validators, and EF mappings together.
- `apps/web` - React 19 + Tailwind 4 SPA; primitives in `src/components`, feature logic in `src/features`, shared hooks in `src/lib`, tokens in `src/styles/tokens.css`.
- `apps/marketing` - Astro marketing site plus FormFeeder snippets; Markdown in `src/content`, reusable forms in `src/components/forms`.
- `packages/` hosts shared validation/telemetry/contracts; 
- `docs/` contains subfolders for planning, specs, guides, sprints, audits, runbooks, and reference material (see `docs/README.md`).

## Build, Test, and Development Commands
- Backend: `dotnet build backend/Identity.Api/Identity.Api.csproj`, `dotnet run --project backend/Identity.Api/Identity.Api.csproj`, `dotnet test backend/Identity.Api.Tests/Identity.Api.Tests.csproj` (adjust paths if the test project differs).
- Identity Host: `dotnet build backend/Identity.Host/Identity.Host.csproj`, `dotnet run --project backend/Identity.Host/Identity.Host.csproj`.
- Web: `npm install`, `npm run dev --workspace apps/web`, `npm run lint --workspace apps/web`, `npm run test --workspace apps/web`.
- Marketing: `npm run dev --workspace apps/marketing` for preview; reuse Playwright smoke tests from `tests/web/e2e`.

## Coding Style & Naming Conventions
- C#: 4-space indent, `PascalCase` types, `camelCase` locals, `Async` suffix on `Task` members; DTOs as `<Feature>Request`/`<Feature>Response`.
- TypeScript/React: 2-space indent, `PascalCase` components, `camelCase` hooks/utilities, `kebab-case` files; prefer functional components and TanStack Query.
- Astro: content collections, `kebab-case` routes, colocated CSS modules.

## Testing Guidelines
- API: xUnit + FluentAssertions; mock external connectors in `backend/Identity.Api.Tests/Integrations` (or equivalent) and cover Identity.Base auth plus connector orchestration.
- Web: Vitest for units and Playwright for journeys; keep specs beside features and end-to-end flows in `tests/web/e2e`.
- Marketing: `npm run astro check` plus shared Playwright funnels; target >=80% coverage on auth, submission, and notification paths.

## Commit & Pull Request Guidelines
- Adopt Conventional Commits tied to roadmap IDs; scope commits per surface (API, web, marketing) and ship migrations/generated clients with their feature.
- PRs need a summary, screenshots for UI/Astro updates, test evidence (`dotnet test`, `npm run test`, Playwright artifacts), and doc/config updates. Merge only with green CI and resolved feedback.

## Non-Negotiable Rules
- **NEVER DELETE FILES YOU DID NOT CREATE OR MODIFY IN THIS SESSION.** If you find an unexpected file or change, stop and ask for guidance instead of removing it.
- Favor minimal, well-scoped edits; leave unrelated files untouched.
- Capture assumptions and uncertainties in your hand-off notes so follow-up work is easy.

## Integration Notes
- **Identity.Base**: Follow `docs/Identity_Base_Integration.md` for setup, expose tenant IDs, signing certs, and redirect URIs via `appsettings.Development.json.example`/`.env.sample`, and seed local accounts with the host command (`dotnet run --project backend/Identity.Host/Identity.Host.csproj -- setup`).
- **FormFeeder**: Follow `docs/FormFeeder_Integration.md` for setup. Expose the active form IDs via env vars and keep domain allow-lists current so submissions avoid 401 errors.
- **MailJet**: Surface `MAILJET_API_KEY`/`MAILJET_SECRET` in `.env` and manifests; log template IDs in `docs/mailjet/`. Attachments inherit FormFeeder limits.
