# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore and Build the Solution

Run the following commands from the root of the solution to confirm a clean restore and build:

```bash
dotnet restore
dotnet build --configuration Release
```

Ensure there are no warnings that could indicate compatibility issues, such as deprecated APIs or nullable reference warnings that may have been suppressed.

---

## 2. Run Unit Tests

Execute the test project to confirm all existing tests pass under the new runtime:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output carefully. Pay attention to:
- Any tests that were previously passing but now fail.
- Any tests marked as skipped or inconclusive that may need to be revisited.

---

## 3. Validate the Data Layer

Since `Bookstore.Data` handles data access, verify the following:

- **Database Migrations**: If Entity Framework Core is in use, confirm that existing migrations are compatible with the new version. Run:
  ```bash
  dotnet ef migrations list --project app/Bookstore.Data
  dotnet ef database update --project app/Bookstore.Data
  ```
- **Connection Strings**: Confirm that connection strings in configuration files (`appsettings.json`) are correct for the target environment.
- **Provider Compatibility**: If the project previously used a Windows-specific database provider (e.g., `System.Data.SqlClient`), confirm it has been replaced with the cross-platform equivalent (`Microsoft.Data.SqlClient`).

---

## 4. Validate the Web Project

Run the web application locally to confirm it starts and functions correctly:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj
```

Check the following:
- The application starts without runtime exceptions.
- All routes and pages load as expected.
- Static assets are served correctly.
- Any authentication or session middleware is functioning properly.

---

## 5. Review Configuration Files

Cross-platform migrations can introduce issues with configuration that are not caught at build time. Review the following:

- **`appsettings.json`**: Ensure all environment-specific settings are present and correct.
- **File Paths**: Check for any hardcoded Windows-style file paths (e.g., `C:\`) in configuration or code that would not be valid on Linux or macOS.
- **Environment Variables**: Confirm that any environment variables referenced in the application are set appropriately in the target environment.

---

## 6. Validate the CDK Project

If `Bookstore.Cdk` defines infrastructure, review the generated output to confirm it reflects the intended infrastructure for the new cross-platform deployment target:

```bash
dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj
```

Verify that the synthesized output matches expectations before applying any infrastructure changes.

---

## 7. Perform a Runtime Smoke Test

After confirming the build and unit tests pass, perform a manual or scripted smoke test against a staging environment to validate end-to-end behavior, including:

- Data retrieval and persistence through `Bookstore.Data`.
- Domain logic execution through `Bookstore.Domain`.
- Web layer responses from `Bookstore.Web`.

---

## 8. Review Target Framework and Dependencies

Confirm the following in each `.csproj` file:

- The `<TargetFramework>` is set to the intended version (e.g., `net8.0`).
- All NuGet package versions are current and do not reference packages that are no longer maintained or have known vulnerabilities. Run:
  ```bash
  dotnet list package --outdated
  dotnet list package --vulnerable
  ```

Address any vulnerable packages before deploying to production.