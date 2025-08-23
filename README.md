# e-v-re Standard GitHub Action Build (composite action)

This composite action automates version bumping, building, testing, packaging, publishing to GitHub Packages (NuGet), tagging, releasing, and generating an SBOM for .NET projects.

What it does (high level)
- Checks out the repo with full history (needed for versioning).
- Installs .NET 8 SDK, optionally installs .NET Core 3.1 or .NET 6.0 SDKs.
- Bumps project version (revision) across all .csproj/.props files.
- Commits the version bump, creates a git tag v{newVersion}, and pushes to master with tags.
- Adds a private GitHub Packages NuGet source for the e-v-re org.
- Restores dependencies and builds the solution (passing the new version).
- Exports provided secrets to environment variables.
- Optionally lists repository files for debugging.
- Runs tests:
  - Full test run, or
  - Limited to Unit and Integration categories, based on input.
- Collects generated .nupkg files and pushes them to GitHub Packages.
- Creates a GitHub Release with the version tag.
- Generates an SBOM and uploads it as an artifact.

Inputs
- secrets (required): JSON string of secrets used in the workflow. Must include at least:
  - NUGET_PACKAGE_TOKEN: token with permission to publish packages and create releases.
- limit_testing (default: false): If true, runs only tests in categories Unit and Integration; otherwise runs all tests.
- debug_directory (default: false): If true, prints a recursive directory listing.
- frameworks (optional): Install an additional SDK. Supported values:
  - 3.1.x
  - 6.0.x

Outputs
- newVersion: The bumped project version (derived from the updater step).

Key implementation details
- Version bump: vers-one/dotnet-project-version-updater@v1.6 updates all .csproj/.props files with a revision bump and exposes newVersion.
- Git operations: Commits the bump, tags v{newVersion}, pushes to master and pushes tags using the GitHub actor identity.
- NuGet source: Adds a GitHub Packages source for org e-v-re and uses NUGET_PACKAGE_TOKEN for authentication.
- Build and packaging: Runs dotnet build with /p:Version set; expects packages to be produced (e.g., GeneratePackageOnBuild=true in projects).
- Test selection: limit_testing toggles between full test run and a filter that includes Category=Unit and Category=Integration.
- Release and SBOM: Creates a GitHub Release for the new tag and uploads an SBOM artifact.

Artifacts and publishing
- NuGet packages: Pushed to GitHub Packages under the “Github e-v-re Private Repo” source.
- GitHub Release: Created with tag v{newVersion}.
- SBOM: Generated and uploaded as an artifact named “SBOM”.

Example usage
```yaml
name: Build and Publish
on:
  push:
    branches: [ master ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Standard build and publish
        uses: e-v-re/phoenix-composite-action@master
        with:
          # Provide only what you need; at minimum include NUGET_PACKAGE_TOKEN
          secrets: |
            {
              "NUGET_PACKAGE_TOKEN": "${{ secrets.NUGET_PACKAGE_TOKEN }}"
            }
          limit_testing: "false"        # or "true"
          debug_directory: "false"      # or "true"
          frameworks: "6.0.x"           # or "3.1.x" or omit
```

Notes
- This action pushes commits and tags directly to the master branch.
- Ensure your projects are configured to produce .nupkg on build or add a separate pack step.
- NUGET_PACKAGE_TOKEN must have permissions to publish packages (write:packages) and create releases (repo).
