# Reusable Workflows for SonarQube Cloud

This repository contains reusable workflows for integrating SonarQube Cloud into your CI/CD pipeline.

> These workflows is part of the Codebelt umbrella and ensures a consistent way of: 
> 
> - Defining your CI/CD pipeline 
> - Structuring your repository
> - Keeping your codebase small and feasible
> - Writing clean and maintainable code
> - Deploying your code to different environments
> - Automating as much as possible
>
> A paved path to excel as a DevSecOps Engineer.

## Available Workflows

- [default.yml](.github/workflows/default.yml) - the default workflow that:
  - [fetches the codebase](https://github.com/codebeltnet/git-checkout),
  - [installs the .NET SDK](https://github.com/codebeltnet/install-dotnet),
  - [installs the SonarScanner for .NET tool](https://github.com/codebeltnet/dotnet-tool-install-sonarscanner),
  - [restores the dependencies](https://github.com/codebeltnet/dotnet-restore),
  - [runs the SonarQube Cloud analysis](https://github.com/codebeltnet/sonarcloud-scan),
  - [builds the solution](https://github.com/codebeltnet/dotnet-build),
  - [finalizes the SonarQube Cloud analysis](https://github.com/codebeltnet/sonarcloud-scan-finalize).

### Usage

To call this workflow in your GitHub repository, you can follow these steps:

```yaml
sonarcloud-call:
    uses: codebeltnet/jobs-sonarcloud/.github/workflows/default.yml@v3
```

### Inputs

```yaml
with:
  # Optional path to the project(s) file to build. Pass empty to have MSBuild use the default behavior. Supports globbing. Default is an empty string.
  projects:
  # Optional checkout branch, tag or SHA. Omit to retain the triggering ref.
  ref: ''
  # Build configuration. Defaults to Debug for existing callers.
  configuration: Debug
  # The name of your organization in SonarQube Cloud.
  organization:
  # The key of your project in SonarQube Cloud.
  projectKey:
  # The version of your project, e.g., 1.0.0.
  version:
  # The URL of your SonarQube instance.
  host: 'https://sonarcloud.io'
  # When set to true, includes preview versions of .NET. Default is false.
  include-preview: false
  # Additional properties to be passed to the scanner.
  parameters: >-
    -d:sonar.exclusions='**/obj/**,**/bin/**'
    -d:sonar.sources='src/'
    -d:sonar.tests='test/'
  # The maximum time in minutes to allow the job to run. Default is 15 minutes.
  timeout-minutes: 15
  # Coverage representation: opencover (default) or normalized (opt in).
  coverage-mode: opencover
  # Repository-relative source directory, used only in normalized mode.
  coverage-source-root: src
  # Artifact-relative capture groups identify build variant and target framework.
  coverage-partition-regex: '^([^/]+?)(?:-[0-9a-f]{16,})?/([^/]+)/'
  # Optional caller acceptance guard; empty means no maximum source-line spread.
  coverage-max-identity-spread: ''
```

These four optional inputs are forwarded to `sonarcloud-scan@v2`. Defaults preserve existing OpenCover consumers. To opt in, set `coverage-mode: normalized` and `coverage-source-root: src`. The workflow's existing .NET installation supplies the stable .NET 10 SDK required by normalization; the action itself installs no prerequisites.

Both modes download the raw `TestResults*` artifacts once. Normalized mode generates and verifies one Sonar generic coverage report before scanner begin, then retains the existing build and finalization sequence. It configures `sonar.coverageReportPaths` and omits `sonar.cs.opencover.reportsPaths`. The default mode retains the existing OpenCover glob; both retain VSTest report ingestion.

The partition pattern matches paths relative to `artifacts` and groups the build variant and target framework. All reports must match; customize the pattern for other artifact layouts. The spread guard is empty by default and belongs to the caller. Both coverage property names are reserved and rejected in `parameters`, including same-mode overrides. Quoted additional arguments are passed literally without shell evaluation.

The normalizer unions execution evidence and deduplicates logical source structure using the available OpenCover identities. Compiler-divergent methods can make ordinal/path correlation ambiguous; this is deterministic aggregation rather than perfect compiler-independent branch identity. Read the [shared normalizer contract and validation provenance](https://github.com/codebeltnet/sonarcloud-scan/blob/main/docs/coverage-normalization.md) before selecting a caller acceptance guard. Codecov remains a separate raw-evidence consumer.

For post-release assurance, set `ref` to the exact released SHA and `configuration: Release`. Use the existing `parameters` input to set `-d:sonar.branch.name=main` and `-d:sonar.scm.revision=<released SHA>`, retaining `-d:sonar.exclusions='**/obj/**,**/bin/**'` because custom parameters replace the defaults. Set `version` to the released SemVer. Omitting the new inputs retains the triggering checkout and Debug build used by existing callers.

### Secrets

```yaml
secrets:
  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Outputs

This workflow has no outputs.

### Example

```yaml
jobs:
  sonarcloud:
    needs: [build,test]
    uses: codebeltnet/jobs-sonarcloud/.github/workflows/default.yml@v3
    with:
      organization: your-sonarcloud-organization
      projectKey: your-sonarcloud-projectkey
      version: ${{ needs.build.outputs.version }}
      include-preview: true
    secrets: inherit
```

## Contributing to Reusable Workflows for SonarQube Cloud

Contributions are welcome! 
Feel free to submit issues, feature requests, or pull requests to help improve these workflows.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
