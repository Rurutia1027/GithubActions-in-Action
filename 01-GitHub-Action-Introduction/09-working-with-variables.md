# Working with Different Level Variables 

### Explanation of Variable Scopes
- **Workflow-level env**: default for all jobs/steps unless overridden. 
- **Job-level env**: overrides or adds variables for that job only. 
- **Step-level env**: we can define with `env: xxx` inside a step if needed. 
- Variables are referenced with `${{ env.VAR_NAME }}` anywhere in the scope of current YAML. 


### Demo YAML Files for GitHub Action Workflow 
```yml 
name: GitHub Action Workflow with Multiple Jobs - Variables Demo with Docker Login

on: 
  push:
    branches:
      - main

env:  # workflow scoped/level variables 
  JAVA_VERSION: '17'
  NODE_VERSION: '18'
  BACKEND_DIR: 'backend'
  FRONTEND_DIR: 'frontend'

  # Docker registry login info (username as env var, password from secrets)
  DOCKER_USERNAME: 'your-docker-username'

jobs:
  build_job_1:
    name: Build Frontend & Backend
    runs-on: ubuntu-latest
    env:  # job scoped/level variables 
      MVN_OPTS: '-DskipTests'
      NODE_VERSION: '16'
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'

      - name: Build Backend
        run: |
          cd ${{ env.BACKEND_DIR }}
          mvn clean install $MVN_OPTS

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Build Frontend
        run: |
          cd ${{ env.FRONTEND_DIR }}
          npm ci
          npm run build

  test_job_2:
    name: Run Automated Tests
    runs-on: ubuntu-latest
    needs: build_job_1
    env:
      MVN_OPTS: ''
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'

      - name: Run Backend Tests
        run: |
          cd ${{ env.BACKEND_DIR }}
          mvn test $MVN_OPTS

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Run Frontend Tests
        run: |
          cd ${{ env.FRONTEND_DIR }}
          npm ci
          npm test -- --watchAll=false

  deploy_job_3:
    name: Deploy Application
    runs-on: ubuntu-latest
    needs: test_job_2
    env:
      DEPLOY_SERVER: 'example.com'
      DEPLOY_USER: 'deployuser'
      DEPLOY_PATH: '/var/www/app'
      DOCKER_USERNAME: ${{ env.DOCKER_USERNAME }}
      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Docker Login
        run: |
          echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

      - name: Deploy to Server (example SCP)
        run: |
          echo "Deploying to $DEPLOY_USER@$DEPLOY_SERVER:$DEPLOY_PATH"
          # scp -r ${{ env.BACKEND_DIR }}/target/*.jar $DEPLOY_USER@$DEPLOY_SERVER:$DEPLOY_PATH
```