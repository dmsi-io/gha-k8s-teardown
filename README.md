# gha-k8s-teardown

The purpose of this GitHub Action is to automate the teardown of Namespaces that have been created with new feature branches. This action will always delete deployments and replace the NodePort service with an ExternalName service of the triggering repo. After this, if the namespace is safe to teardown it will be deleted.

### Usage

```yaml
name: Teardown Namespace

on: delete

jobs:
  teardown-namespace:
    name: Teardown Namespace
    runs-on: ubuntu-latest
    steps:
      - name: Teardown Kubernetes Namespace
        uses: dmsi-io/gha-k8s-teardown@v1
        with:
          GCP_SA_KEY: ${{ secrets.GCP_SA_KEY }}
          GKE_CLUSTER_NAME: ${{ secrets.GCP_STAGING_CLUSTER_NAME }}
          GCP_ZONE: ${{ secrets.GCP_ZONE }}
          GCP_PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
          GHA_ACCESS_USER: ${{ secrets.GHA_ACCESS_USER }}
          GHA_ACCESS_TOKEN: ${{ secrets.GHA_ACCESS_TOKEN }}
```

> It is recommended create this GHA in its own yaml file with `delete` as the only trigger. This will only be ran if the action exists within the [default branch](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows#delete).

### Optional inputs

#### Repositories

Sometimes it is important to actually deploy a copy of an application to the feature branch namespace. For our architecture, the primary contender is `graphql-api` for our middleware. Because of the namespace check, it is recommended to supply these types of dependencies in the `repo` input. This will allow that repo to be checked for an open branch of the same name, if there is no branch then the namespace will be considered safe to delete.

```yaml
  with:
    repos: graphql-api

  # multiple dependencies can be supplied as a comma-separated list
  with:
    repos: graphql-api, docs-api
```

#### From Namespace

This value shouldn't be overridden under most circumstances, but is provided on the off chance it is required. Providing this will determine the namespace to grab resources from:

Default: `${{ github.event.repository.default_branch }}`

```yaml
  with:
    FROM_NAMESPACE: main
```