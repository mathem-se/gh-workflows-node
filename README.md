# Reusable workflows

## Contribute

After you have merged your PR to **develop** you can test it out in your repo by pointing the workflow towards **@develop** (`uses: mathem-se/gh-workflows-node/.github/workflows/pull-request.yml@develop`). When you verified it worked you can "release it" to all users of these workflows by merging to **main**. (don't forget to re-point your repo to **@main**).

## Getting started

Make sure your repo has the following scripts:

For jest:
`"test:ci": "jest --coverage --ci --json --testLocationInResults --outputFile=./coverage/report.json"`

For vitest:

`"test:ci": "vitest run --coverage --reporter=json --outputFile.json=./coverage/report.json"`

### Pull requests

Add the following file `.github/workflows/pull-request.yml` to your project with this content:

```yaml
name: Pull request
on: [pull_request]

jobs:
  PR:
    uses: mathem-se/gh-workflows-node/.github/workflows/pull-request.yml@main
    with:
      CHECK_LINT: false #if you don't want to check lint (defaults to true)
    secrets:
      NODE_AUTH_TOKEN: ${{ secrets.GH_PACKAGE_READ_ONLY}}
```

### Deploy to AWS with SAM

Add the following file `.github/workflows/deploy.yml` to your project with this content:

```yaml
name: Deploy to AWS
on:
  push:
    branches: [main, master, develop]

permissions:
  id-token: write
  contents: read

jobs:
  Deploy:
    uses: mathem-se/gh-workflows-node/.github/workflows/deploy-aws.yml@main
    with:
      domain: <repo name> #change value
      team: <team name> #change value
    secrets:
      NODE_AUTH_TOKEN: ${{ secrets.GH_PACKAGE_READ_ONLY }}
      AWS_CICD_ACCOUNT: ${{ secrets.AWS_CICD_ACCOUNT }}
      AWS_CICD_API_URL: ${{ secrets.AWS_CICD_API_URL }}
      AWS_CICD_API_REGION: ${{ secrets.AWS_CICD_API_REGION }}
```

### Deploy new GH package

Add the following file `.github/workflows/deploy.yml` to your project with this content:

```yaml
name: Release Version
on:
  push:
    branches: [main, master, develop]

permissions:
  id-token: write
  contents: write
  issues: write
  pull-requests: write
  packages: write
  checks: write

jobs:
  Deploy:
    uses: mathem-se/gh-workflows-node/.github/workflows/deploy-gh-package.yml@main
    with:
      domain: <repo name> #change value
    secrets:
      NODE_AUTH_TOKEN: ${{ secrets.GH_PACKAGE_READ_ONLY }}
      SEMANTIC_RELEASE_TOKEN: ${{ secrets.GH_TOKEN_SEMANTIC_RELEASE }}
      AWS_CICD_ACCOUNT: ${{ secrets.AWS_CICD_ACCOUNT }}
      AWS_CICD_API_URL: ${{ secrets.AWS_CICD_API_URL }}
      AWS_CICD_API_REGION: ${{ secrets.AWS_CICD_API_REGION }}
```

### Specify custom node-version

All workflows support `node-version` as an input. Default version will be 16.
To run with node 18 use the following input:

```yaml
name: Pull request
on: [pull_request]

jobs:
  PR:
    uses: mathem-se/gh-workflows-node/.github/workflows/pull-request.yml@main
    with:
      CHECK_LINT: false #if you don't want to check lint (defaults to true)
      node-version: 18
    secrets:
      NODE_AUTH_TOKEN: ${{ secrets.GH_PACKAGE_READ_ONLY}}
```

### Specify custom npm run command

Some workflows support `run-command` as an input. This command will be run in the release-step before the "normal" release flow.

```yaml
jobs:
  Deploy:
    uses: mathem-se/gh-workflows-node/.github/workflows/deploy-aws.yml@main
    with:
      domain: 'ecom-service-asdf'
      team: 'ecom-pro'
      run-command: 'build'
      node-version: 18

    secrets:
      NODE_AUTH_TOKEN: ${{ secrets.GH_PACKAGE_READ_ONLY }}
      AWS_CICD_ACCOUNT: ${{ secrets.AWS_CICD_ACCOUNT }}
      AWS_CICD_API_URL: ${{ secrets.AWS_CICD_API_URL }}
      AWS_CICD_API_REGION: ${{ secrets.AWS_CICD_API_REGION }}
```

### Deploy static assets to AWS S3

#### Prerequisites

Make sure that your cloudformation `template.yaml` creates the infra needed to be able to upload the static assets to AWS. A S3 bucket named `<AWS_ACCOUNT_ID>-<REPOSITORY_NAME>` is **required**. You could also optionally sets up a AWS Cloudfront distribution in front off S3 and invalidate the cache using this workflow.

Add the following file `.github/workflows/deploy.yml` to your project with this content:

```yaml
name: Deploy static assets
on:
  push:
    branches: [develop, main, master]

permissions:
  id-token: write
  contents: read

jobs:
  DeployInfra:
    uses: mathem-se/gh-workflows-node/.github/workflows/deploy-aws.yml@main
    with:
      domain: <repo name> #change value
      team: <team name> #change value
    secrets:
      NODE_AUTH_TOKEN: ${{ secrets.GH_PACKAGE_READ_ONLY }}
      AWS_CICD_ACCOUNT: ${{ secrets.AWS_CICD_ACCOUNT }}
      AWS_CICD_API_URL: ${{ secrets.AWS_CICD_API_URL }}
      AWS_CICD_API_REGION: ${{ secrets.AWS_CICD_API_REGION }}
  
  Release:
    needs: [DeployInfra]
    uses: mathem-se/gh-workflows-node/.github/workflows/deploy-static-assets.yml@develop
    with:
      LOCAL_PATH: <local-path> # Local path to static assets to be uploaded, defaults to ./dist
      DISTRIBUTION_ID: <distribution-id> # Optionally provide you cloudfront distribution id to invalidate the cache.
    secrets:
      NODE_AUTH_TOKEN: ${{ secrets.GH_PACKAGE_READ_ONLY }}
      AWS_CICD_ACCOUNT: ${{ secrets.AWS_CICD_ACCOUNT }}
      AWS_CICD_API_URL: ${{ secrets.AWS_CICD_API_URL }}
      AWS_CICD_API_REGION: ${{ secrets.AWS_CICD_API_REGION }}
```
