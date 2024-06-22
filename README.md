# phoenix-composite-action

This workflow is designed to run the standard package creation process which includes the following steps:

1. Checking out the repository.
2. Setting up .NET.
3. Setting up the version(s) for csproj and props file
   a. This increments the build version by +1, e.g., 8.0.0.1 will be 8.0.0.2
4. Updating GitHub version to the new version.
5. Adding a private e-v-re NuGet Registry.
6. Restoring dependencies.
7. Building and Packing.
8. Testing.
9. Uploading the NuGet package to GitHub, then downloading it as an artifact.
10. Pushing the package to GitHub packages under e-v-re.
11. Creates a GitHub release.

It requires two inputs: NuGet Package Token and Repository_Name, and it outputs a newVersion. Action uses the composite method for running the workflow steps. The major purpose of this workflow is to automate packaging and delivery of .NET applications through the NuGet ecosystem.
