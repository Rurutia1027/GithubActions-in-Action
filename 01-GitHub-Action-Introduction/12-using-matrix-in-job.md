# Matrix in Github Action 

The matrix concept was primarily introduced to:
- Automatically generate test jobs for multiple version or environment combinations,
- Avoid requiring users to manually write repetitive workflow configurations,
- Allow users to simply define the “version dimensions,” and let GitHub Actions run all combinations for them.


## Summary of matrix concepts 
- strategy: Defines how to run multiple job variants 
- matrix: Lists combinations of variables for each job run
- `${{ matrix.VAR }}`: Access current matrix variable value inside job steps 
- Parallel jobs: Matrix jobs run in parallel, speeding up testing across variations

## What this YML does ? 

- Runs 4 job in parallel (2 envs * 2 Node.js versions)
- Each job picks the right env and node_version from the matrix 
- concurrency group cancels previous jobs on the same workflow + branch when new ones start
- env: set dynamically per matrix, useful if we wanna to deploy to test differently 

```
name: Demo Workflow Using Different GitHub Secrets

on:
  push:
    branches:
      - main

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest

    # Concurrency controls job concurrency per workflow+branch
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
      cancel-in-progress: true

    # Matrix defines multiple job variations
    strategy:
      matrix:
        environment: [production, staging]        # Different deployment envs
        node_version: [16, 18]                   # Different Node.js versions

    environment: ${{ matrix.environment }}

    env:
      REPO_API_KEY: ${{ secrets.REPO_API_KEY }}
      ORG_CLOUD_TOKEN: ${{ secrets.ORG_CLOUD_TOKEN }}
      PROD_DB_PASSWORD: ${{ secrets.PROD_DB_PASSWORD }}
      DOCKER_USERNAME: 'my-docker-user'
      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node_version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node_version }}

      - name: Login to Docker Registry
        run: |
          echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

      - name: Use Organization-level secret for Cloud CLI
        run: |
          echo "Using cloud token: $ORG_CLOUD_TOKEN in ${{ matrix.environment }}"

      - name: Connect to DB using environment secret
        run: |
          echo "Connecting to DB in ${{ matrix.environment }} with password $PROD_DB_PASSWORD"
```


## What do include and exclude on in the scope of matrix ? 

#### exclude 
Removes specific combinations from the automatically generated full matrix. 
Use it to skip invalid, unsupported, or unnecessary combinations that don't make sense or won't work.

#### include 
Adds specific combinations outside the automatic cross-product of the matrix lists. 
Use it to add specifical or edge cases configurations you want to test that don't fit the main matrix pattern. 
