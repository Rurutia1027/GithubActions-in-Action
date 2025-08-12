# GitHub Action Context Concept and Usage Demo 

## Context Use Demo 
```yaml 
name: Demo Workflow Using Contexts

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

    steps:
      - name: Show GitHub Context
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch/Ref: ${{ github.ref }}"
          echo "Commit SHA: ${{ github.sha }}"
          echo "Actor (who triggered the workflow): ${{ github.actor }}"

      - name: Show Job Context
        run: |
          echo "Job status: ${{ job.status }}"
          echo "Runner OS: ${{ runner.os }}"
          echo "Runner name: ${{ runner.name }}"

      - name: Show Environment Context
        run: |
          echo "Workflow name: ${{ workflow }} (Note: available via github.workflow)"
          echo "Run ID: ${{ github.run_id }}"
          echo "Run number: ${{ github.run_number }}"
```

## What are Contexts in GitHub Actions ? 

A context is a way to access information about the workflow run, jobs, steps, environments, and more in GitHub Actions. They are written using the expression syntax: 

```yaml 
${{ context_name.property }}
```

## Common Context Types 
#### github 
It provides info about the repo, branch, commit, event, etc. 
Use it like this `${{ github.ref }}` -> refers/heads/main 

#### env 
It provides info about the environment variables you set in env: blocks. 
Use it like this `${{env.MY_VAR }}`


#### job 
It provides info about the current job. 
Use it like this `${{ job.status }}`

#### runner 
It provides info about the runner machine. 
Use it like this `${{ runner.os }}`

#### steps 
It provides info about the output from previous steps.
Use it like this `${{ steps.step_id.outputs.key }}`

#### secrets 
It provides info about sensitive values from repo/org/env secrets. 
Use it like this `${{ steps.step_id.outputs.key }}`
