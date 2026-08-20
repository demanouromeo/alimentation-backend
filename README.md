# AlimentationDongmo.Api

ASP.NET Core 8 Web API for **Alimentation Dongmo** — inventory and point-of-sale management for a single grocery store in Yaoundé, Cameroun. Currency is FCFA (XAF); domain names and API responses are in French.

This is the **local development** copy of the backend. A separate copy deployed to Railway lives in `../backend-railway`.

## Architecture

Layered solution (`AlimentationDongmo.slnx`):

- **`AlimentationDongmo.Domain`** — entities (`Produit`, `Vente`, `LigneVente`, `Fournisseur`, `CommandeFournisseur`, `LigneCommandeFournisseur`, `Categorie`, `MouvementStock`, `ProduitFournisseur`) and enums, no external dependencies.
- **`AlimentationDongmo.Infrastructure`** — EF Core `DbContext` (Pomelo MySQL provider), ASP.NET Core Identity setup, roles.
- **`AlimentationDongmo.Api`** — controllers, JWT auth, Program.cs composition root.
- **`AlimentationDongmo.Seeder`** — standalone console project that seeds dev data (roles, an admin user, sample products/suppliers) into the configured database.

### Controllers

`AuthController`, `ProduitsController`, `CategoriesController`, `FournisseursController`, `CommandesFournisseurController`, `VentesController`, `UtilisateursController`, `DashboardController` — all under `src/AlimentationDongmo.Api/Controllers`.

### Auth & roles

ASP.NET Core Identity + JWT bearer tokens. Three roles (`Infrastructure/Identity/Roles.cs`):

- `Administrateur`
- `Caissier`
- `Approvisionnement`

Endpoints are gated with `[Authorize(Roles = ...)]`.

## Requirements

- .NET 8 SDK
- MySQL 8 running locally, reachable at `127.0.0.1:3306`

## Running locally

```bash
cd src/AlimentationDongmo.Api
dotnet run
```

Listens on `http://localhost:5144` (see `Properties/launchSettings.json`; `https://localhost:7046` is also available under the `https` launch profile). Swagger UI opens automatically at `/swagger`.

This runs via plain `dotnet run`, **not** `dotnet watch` — controller/code changes require killing the process and re-running; hot reload does not pick them up.

### Database

Connection string (`appsettings.json`):

```
Server=127.0.0.1;Port=3306;Database=alimentation;User Id=root;Password=;
```

No password locally. Create the `alimentation` schema and apply EF Core migrations before first run:

```bash
cd src/AlimentationDongmo.Api
dotnet ef database update
```

### Seeding dev data

```bash
cd src/AlimentationDongmo.Seeder
dotnet run
```

Uses its own `appsettings.json` (same local connection string by default).

## Configuration

`appsettings.json` sections:

- `ConnectionStrings:DefaultConnection` — MySQL connection string.
- `Jwt` — issuer, audience, signing key, access token lifetime (minutes). The committed key is a dev-only placeholder.
- `Smtp` — outgoing mail (password reset, etc.), unset by default in dev.
- `Frontend:BaseUrl` — used to build links back to the SPA (e.g. password reset), defaults to `http://localhost:5173`.
- `Cors:AllowedOrigins` — origins allowed to call the API. In dev, `Program.cs` also allows any `localhost` origin regardless of port, so the Vite dev server and locally-run desktop/mobile shells work without extra config. Notable non-localhost origins already listed: `https://alimentation.dmsacad.com` (production frontend), `https://localhost` (Capacitor Android WebView), `https://tauri.localhost` (Tauri desktop WebView2).

## Verification

```bash
dotnet build
```

Run from `backend/` (or point at a specific `.csproj`). If `dotnet run` is already active, build to a scratch output directory (`-o <tmp-path>`) to avoid file-lock errors instead of stopping the running server.

## Deployment

This copy is for local development only. Production runs from `../backend-railway`, deployed to Railway as service `alimentation-service`. See that folder and the root `CLAUDE.md` for deployment specifics (env-var config, migration steps, CORS caveats).
