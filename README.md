# Reusable workflows

## Getting started

Add the following file `.github/workflows/pull-request.yml` to your project with this content:

```yaml
name: Pull request
on: [pull_request]

jobs:
  pull_request:
    uses: mathem-se/ecom-gh-actions/.github/workflows/pull-request.yml@main
    secrets:
      NODE_AUTH_TOKEN: ${{ secrets.GH_PACKAGE_READ_ONLY}}
```

Make sure your repo has the following scripts:

For jest:
`"test:ci": "jest --coverage --ci --json --testLocationInResults --outputFile=./coverage/report.json"`

For vitest:

`"test:ci": "vitest run --coverage --reporter=json --outputFile.json=./coverage/report.json"`
