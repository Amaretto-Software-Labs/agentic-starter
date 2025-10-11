# Identity.Base Integration Guide

## Overview
Identity.Base delivers a complete OpenID Connect authority on top of ASP.NET Core Identity, OpenIddict, MFA, and external providers. Consume the published NuGet packages to supply authentication for the `.NET 9` API, React 19 SPA, and Astro marketing site without cloning the upstream repository. Upstream docs: https://github.com/Amaretto-Software-Labs/identity-base.

## Step 1 - Host Setup
1. Create (or reuse) an ASP.NET Core host project that follows .NET naming conventions (PascalCase project/folder). For example:
   ```bash
   mkdir backend
   cd backend
   dotnet new web -n Identity.Host
   ```
   Keep the solution file in the monorepo root and place .NET projects under `backend/`. Each .NET project lives in its own PascalCase folder (for example `backend/Identity.Host`, `backend/<Project Name>.Api`). React/Astro apps stay under `apps/`.
2. Install the core package:
   ```bash
   dotnet add backend/Identity.Host/Identity.Host.csproj package Identity.Base
   ```
3. Replace `Program.cs` with the minimal composition:
   ```csharp
   using Identity.Base.Extensions;

   var builder = WebApplication.CreateBuilder(args);
   builder.Services.AddIdentityBase(builder.Configuration, builder.Environment);

   var app = builder.Build();
   app.UseApiPipeline();
   app.MapControllers();
   app.MapApiEndpoints();
   await app.RunAsync();
   ```

## Step 2 - Configuration
Add or update `appsettings.Development.json` with the required sections:
```json
{
  "ConnectionStrings": {
    "Primary": "User ID=postgres;Password=P@ssword123;Host=localhost;Port=5432;Database=identity"
  },
  "IdentitySeed": {
    "Enabled": true,
    "Email": "admin@example.com",
    "Password": "Passw0rd!Passw0rd!",
    "Roles": ["IdentityAdmin"]
  },
  "Registration": {
    "ConfirmationUrlTemplate": "https://localhost:5001/account/confirm?token={token}&email={email}",
    "PasswordResetUrlTemplate": "https://localhost:5001/reset-password?token={token}&email={email}",
    "ProfileFields": [
      { "Name": "displayName", "DisplayName": "Display Name", "Required": true, "MaxLength": 128 }
    ]
  },
  "Cors": {
    "AllowedOrigins": ["https://localhost:5173", "http://localhost:5173"]
  },
  "MailJet": {
    "FromEmail": "noreply@example.com",
    "FromName": "Identity Base",
    "ApiKey": "<mailjet-key>",
    "ApiSecret": "<mailjet-secret>",
    "Templates": {
      "Confirmation": 123456,
      "PasswordReset": 234567,
      "MfaChallenge": 345678
    }
  },
  "Mfa": {
    "Issuer": "Identity Base",
    "Email": { "Enabled": true },
    "Sms": { "Enabled": false }
  },
  "ExternalProviders": {
    "Google": { "Enabled": false },
    "Microsoft": { "Enabled": false },
    "Apple": { "Enabled": false }
  },
  "OpenIddict": {
    "ServerKeys": { "Provider": "Development" },
    "Applications": [
      {
        "ClientId": "spa-client",
        "ClientType": "public",
        "RedirectUris": ["http://localhost:5173/auth/callback"],
        "Permissions": [
          "endpoints:authorization",
          "endpoints:token",
          "endpoints:userinfo",
          "grant_types:authorization_code",
          "response_types:code",
          "scopes:openid",
          "scopes:profile",
          "scopes:email",
          "scopes:offline_access",
          "scopes:<Project Name>.Api"
        ],
        "Requirements": ["requirements:pkce"]
      }
    ]
  }
}
```
Key actions:
- Supply a PostgreSQL connection string for `AppDbContext`.
- Configure MailJet credentials or the host will fail on startup.
- Register SPA/API clients under `OpenIddict:Applications`.
- Use `IdentitySeed` to bootstrap an administrator for local work.

Run migrations and start the host:
```bash
dotnet ef database update \
  --project backend/Identity.Host/Identity.Host.csproj \
  --startup-project backend/Identity.Host/Identity.Host.csproj

dotnet run --project backend/Identity.Host/Identity.Host.csproj
```

## Step 3 - Connect Consumers
- **API (`backend/<Project Name>.Api`)**: install `Identity.Base.AspNet` (`dotnet add backend/<Project Name>.Api/<Project Name>.Api.csproj package Identity.Base.AspNet`), call `AddIdentityBaseAuthentication("https://localhost:5001")`, and require scopes (e.g. `<Project Name>.Api`).
- **React SPA**: configure PKCE client IDs, redirect URIs, and scopes to match the identity host. Use the discovery document (`/.well-known/openid-configuration`) for client libs.
- **Astro marketing**: ensure the domain appears in both Identity.Base CORS and FormFeeder allow lists when embedding login forms.

## Optional Packages
- **Identity.Base.Roles** adds role/permission definitions and `/users/me/permissions`. Install `Identity.Base.Roles`, call `AddIdentityRoles`, add the `Permissions`/`Roles` sections, and create migrations with the `IdentityRolesDbContext` context.
- **Identity.Base.Admin** registers full admin endpoints (`/admin/users`, `/admin/roles`) and includes roles support. Install `Identity.Base.Admin`, call `AddIdentityAdmin`, and expose the admin scope to clients.

## Operations & Testing
- Execute `dotnet test Identity.sln` within the Identity Base repo before upgrading packages.
- Use `/healthz` for readiness checks (database, MailJet, external providers).
- Switch `OpenIddict:ServerKeys:Provider` to `File` or `AzureKeyVault` before production.
- After package upgrades, re-test key flows: registration, login, MFA, password reset, external providers, admin APIs (if enabled).

For deeper configuration examples, follow the upstream [Getting Started with Identity Base Packages](https://github.com/Amaretto-Software-Labs/identity-base/blob/main/docs/guides/getting-started.md) guide.
