# GHA Cloud Run Deploy

[![actions-workflow-main][actions-workflow-main-badge]][actions-workflow-main]
[![release][release-badge]][release]

This GitHub Action is a composite action that facilitates deployments and feature branch deployments to Google Cloud Run.

This action is dependent on a valid Dockerfile existing at the top level of the acting repository. The example in this repo is a simple nginx image, but it is expected that repos using this action will perform their own prior build steps and rely on this action to build and publish the Docker image.

Normal push commit triggers will deploy the application as the new latest branch and will receive all traffic. Because of this, it recommended to only run this action for push commits to the default branch of the repo.

To make use of the feature branches, use this action with a pull_request trigger. This will allow the Cloud Run deployment to add a new tagged release corresponding to the PR number and will also provide a deployment URL that can be commented onto the PR.

> Examples of this can be found in: [.github/workflows/main.yml](.github/workflows/main.yml).

## Inputs

| NAME                    | DESCRIPTION                              | TYPE      | REQUIRED | DEFAULT  |
| ----------------------- | ---------------------------------------- | --------- | -------- | -------- |
| `GCP_IDENTITY_PROVIDER` | GCP Workload Identity Provider.          | `string`  | `true`\* |          |
| `GCP_SERVICE_ACCOUNT`   | GCP Service Account email.               | `string`  | `true`\* |          |
| `GCP_SA_KEY`            | GCP Service Account Key (JSON).          | `string`  | `true`\* |          |
| `GCP_PROJECT_ID`        | GCP Project ID.                          | `string`  | `true`   |          |
| `REGISTRY_HOSTNAME`     | Hostname of Container Registry.          | `string`  | `false`  | `gcr.io` |
| `port`                  | Port of Docker image to receive traffic. | `integer` | `false`  | `80`     |

> It is recommended to use Workload Identity Federation with the `GCP_IDENTITY_PROVIDER` and `GCP_SERVICE_ACCOUNT` inputs. `GCP_SA_KEY` will still work with `v1` tags.

## Outputs

| NAME  | DESCRIPTION                          | TYPE     |
| ----- | ------------------------------------ | -------- |
| `url` | The URL to the deployed application. | `string` |

## Example

```yaml
name: Google Cloud Run Revision Deploy
on:
  pull_request:
  push:
    branches:
      - main

jobs:
  build-deploy:
    name: Build and Deploy to Cloud Run
    runs-on: ubuntu-latest

    permissions:
      contents: 'read'
      id-token: 'write'

    steps:
      - name: Checkout Repo
        uses: actions/checkout@v2

      # any pre-docker build steps will be placed here

      - name: Run self
        uses: dmsi-io/gha-cloudrun-deploy@main
        id: deploy
        with:
          GCP_IDENTITY_PROVIDER: ${{ secrets.GCP_IDENTITY_PROVIDER }}
          GCP_SERVICE_ACCOUNT: ${{ secrets.GCP_SERVICE_ACCOUNT }}
          GCP_PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
```

> Workload Identity Federation requires access to the id-token permission and thus the outlined permissions in the example above are required.

#### With Service Account Credentials JSON

```yaml
name: Google Cloud Run Revision Deploy
on:
  pull_request:
  push:
    branches:
      - main

jobs:
  build-deploy:
    name: Build and Deploy to Cloud Run
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repo
        uses: actions/checkout@v2

      # any pre-docker build steps will be placed here

      - name: Run self
        uses: dmsi-io/gha-cloudrun-deploy@main
        id: deploy
        with:
          GCP_SA_KEY: ${{ secrets.GCP_SA_KEY }}
          GCP_PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
```

For a further practical example, see [.github/workflows/main.yml](.github/workflows/main.yml).

<!-- badge links -->

[actions-workflow-main]: https://github.com/dmsi-io/gha-cloudrun-deploy/actions/workflows/main.yml
[actions-workflow-main-badge]: https://img.shields.io/github/workflow/status/dmsi-io/gha-cloudrun-deploy/Google%20Cloud%20Run%20Revision%20Deploy?label=Google%20Cloud%20Run%20Revision%20Deploy&style=for-the-badge&logo=github
[release]: https://github.com/dmsi-io/gha-cloudrun-deploy/releases
[release-badge]: https://img.shields.io/github/v/release/dmsi-io/gha-cloudrun-deploy?style=for-the-badge&logo=github
