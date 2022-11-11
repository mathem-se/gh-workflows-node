# Reusable workflows

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
