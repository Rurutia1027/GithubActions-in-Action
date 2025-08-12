# Job Concurrency in GitHub Action Workflow 

## Explanation of Concurrency in Action Workflow 
- group: Defines a unique concurrency group per workflow + branch (so runs on different branches don't block each other)
- cancel-in-progress: true: automatically cancels any currently running job in the same group when a new one starts (useful to avoid stale builds)

## Github Action Demo with Concurrency 

```yml 
name: Demo Workflow Using Different GitHub Secrets

on:
  push:
    branches:
      - main

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest

    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
      cancel-in-progress: true

    environment: production  

    env:
      REPO_API_KEY: ${{ secrets.REPO_API_KEY }}
      ORG_CLOUD_TOKEN: ${{ secrets.ORG_CLOUD_TOKEN }}
      PROD_DB_PASSWORD: ${{ secrets.PROD_DB_PASSWORD }}
      DOCKER_USERNAME: 'my-docker-user'
      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to Docker Registry
        run: |
          echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

      - name: Use Organization-level secret for Cloud CLI
        run: |
          echo "Using cloud token: $ORG_CLOUD_TOKEN"

      - name: Connect to Production DB using environment secret
        run: |
          echo "Connecting to Prod DB with password $PROD_DB_PASSWORD"
```