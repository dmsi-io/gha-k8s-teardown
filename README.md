# gha-k8s-teardown

[![release][release-badge]][release]

The purpose of this GitHub Action is to automate the teardown of Namespaces that have been created with new feature branches. This action will always delete deployments and replace the NodePort service with an ExternalName service of the triggering repo. After this, if the namespace is safe to teardown it will be deleted.

## Setup

This action is reliant on a Service Account with the following permissions:

- Kubernetes Engine Admin
- Storage Admin

Additionally, it is recommended to use Workload Identity Federation. If this is not setup follow the steps here: https://github.com/google-github-actions/auth#setup

## Inputs

| NAME                    | DESCRIPTION                                                                              | TYPE     | REQUIRED | DEFAULT                                         |
| ----------------------- | ---------------------------------------------------------------------------------------- | -------- | -------- | ----------------------------------------------- |
| `GCP_IDENTITY_PROVIDER` | GCP Workload Identity Provider.                                                          | `string` | `true`\* |                                                 |
| `GCP_SERVICE_ACCOUNT`   | GCP Service Account email.                                                               | `string` | `true`\* |                                                 |
| `GCP_SA_KEY`            | GCP Service Account Key (JSON).                                                          | `string` | `true`\* |                                                 |
| `GKE_CLUSTER_NAME`      | Google Kubernetes Engine Cluster name.                                                   | `string` | `true`   |                                                 |
| `GCP_ZONE`              | GCP Zone.                                                                                | `string` | `true`   |                                                 |
| `GCP_PROJECT_ID`        | GCP Project ID.                                                                          | `string` | `true`   |                                                 |
| `FROM_NAMESPACE`        | Allows to override the desired FROM_NAMESPACE variable.                                  | `string` | `false`  | `${{ github.event.repository.default_branch }}` |
| `repos`                 | Comma separated list of repositories to instead deploy a copy from the default namespace | `string` | `false`  |                                                 |

> It is recommended to use Workload Identity Federation with the `GCP_IDENTITY_PROVIDER` and `GCP_SERVICE_ACCOUNT` inputs. `GCP_SA_KEY` will still work with `v1` tags.

### Usage

```yaml
name: Teardown Namespace

on:
  - delete

jobs:
  teardown-namespace:
    name: Teardown Namespace
    runs-on: ubuntu-latest

    permissions:
      contents: 'read'
      id-token: 'write'

    steps:
      - name: Teardown Kubenetes Namespace
        uses: dmsi-io/gha-k8s-teardown@v1
        with:
          GCP_IDENTITY_PROVIDER: ${{ secrets.GCP_IDENTITY_PROVIDER }}
          GCP_SERVICE_ACCOUNT: ${{ secrets.GCP_SERVICE_ACCOUNT }}
          GKE_CLUSTER_NAME: ${{ secrets.GCP_CLUSTER_NAME }}
          GCP_ZONE: ${{ secrets.GCP_ZONE }}
          GCP_PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
          GHA_ACCESS_USER: ${{ secrets.GHA_ACCESS_USER }}
          GHA_ACCESS_TOKEN: ${{ secrets.GHA_ACCESS_TOKEN }}
```

> Workload Identity Federation requires access to the id-token permission and thus the outlined permissions in the example above are required.

> It is recommended to create this GHA in its own yaml file with `delete` as the only trigger. This will only be ran if the action exists within the [default branch](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows#delete).

#### With Service Account Credentials JSON

```yaml
name: Teardown Namespace

on:
  - delete

jobs:
  teardown-namespace:
    name: Teardown Namespace
    runs-on: ubuntu-latest

    steps:
      - name: Teardown Kubenetes Namespace
        uses: dmsi-io/gha-k8s-teardown@v1
        with:
          GCP_SA_KEY: ${{ secrets.GCP_SA_KEY }}
          GKE_CLUSTER_NAME: ${{ secrets.GCP_CLUSTER_NAME }}
          GCP_ZONE: ${{ secrets.GCP_ZONE }}
          GCP_PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
          GHA_ACCESS_USER: ${{ secrets.GHA_ACCESS_USER }}
          GHA_ACCESS_TOKEN: ${{ secrets.GHA_ACCESS_TOKEN }}
```

<!-- badge links -->

[release]: https://github.com/dmsi-io/gha-k8s-teardown/releases
[release-badge]: https://img.shields.io/github/v/release/dmsi-io/gha-k8s-teardown?style=for-the-badge&logo=github
