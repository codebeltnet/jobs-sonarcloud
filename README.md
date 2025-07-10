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
    uses: codebeltnet/jobs-sonarcloud/.github/workflows/default.yml@v1
```

### Inputs

```yaml
with:
  # Optional path to the project(s) file to build. Pass empty to have MSBuild use the default behavior. Supports globbing. Default is an empty string.
  projects:
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
```

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
    uses: codebeltnet/jobs-sonarcloud/.github/workflows/default@v2
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
