# Repository Secrets 
## Types of Secrets in GitHub 
### Repository Level Secrets 
- Stored within a single repository 
- Accessible only to workflows running in that repository
- Examples: API keys, Docker passwords, cloud credentials for that repo's deployment
- Best for secrets specific to one project

### Organization Level Secrets 
- Stored at the organization level 
- Can be configured to be available to multiple repositories within that org. 
- Useful when many repos share the same credentials, e.g., a shared Docker registry, cloud provider keys. 
- Can be restricted by repository or team for granular access. 


### Environment Secrets 
- GitHub Environments are deployment tagets (like staging, production)
- Secrets attached to an environment are only accessible to workflows that deploy to that environment
- Good for separating secrets by deployment stage - e.g., separate DB passwords for staging vs. production 
- Allows manual approvals before deployment to certain environments 

### Dependabot Secrets 
- Used by GitHub's Dependabot to update dependencies securely.
- These secrets are managed separately and sepcifically for Dependabot's access 


--- 

## Best Practices for Using GitHub Secrets 
### Principle 1: Use the narrowest scope possible 
- Don't store all secrets at the org level if only one repo needs them. 
- Use environment secrets for deployment-related keys that differ by environment 
- Limit exposure to the minimum necessary repos or environments.


### Principle 2: Use environment secrets for deployment 
- Create staging, production, qa environments.
- Assign secrets specific to those environments. 
- Enables approval gates and reduces risk of leaking prod secrets to staing builds. 


### Principle 3: Use descriptive, consistent naming 
Examples:
- `DOCKER_NAME, DOCKER_PASSWORD`
- `AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY`
- `PROD_DB_PASSWORD, STAGING_DB_PASSWORD`

### Principle 4: Never commit secrets in code 

### Principle 5: Rotate secrets regularly 

### Principle 6: Use secret scaning 

--- 

## Demo of Using Different Leve Secrets in GitHub Action YML File

```yml 
name: Demo Workflow Using Different GitHub Secrets

on:
  push:
    branches:
      - main

env:
  # Repo-level secret username as env variable
  DOCKER_USERNAME: 'my-docker-user'  

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest

    # Specify environment to enable environment secrets access
    environment: production  

    env:
      # Repo-level secret (must be created under repo secrets)
      REPO_API_KEY: ${{ secrets.REPO_API_KEY }}

      # Organization-level secret (must be granted to this repo)
      ORG_CLOUD_TOKEN: ${{ secrets.ORG_CLOUD_TOKEN }}

      # Environment-level secret (only available because environment=production)
      PROD_DB_PASSWORD: ${{ secrets.PROD_DB_PASSWORD }}

      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to Docker Registry (using repo-level username and secret)
        run: |
          echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

      - name: Use Organization-level secret for Cloud CLI
        run: |
          echo "Using cloud token: $ORG_CLOUD_TOKEN"
          # cloud-cli login --token $ORG_CLOUD_TOKEN

      - name: Connect to Production DB using environment secret
        run: |
          echo "Connecting to Prod DB with password $PROD_DB_PASSWORD"
          # ./connect-db.sh --password "$PROD_DB_PASSWORD"
```
