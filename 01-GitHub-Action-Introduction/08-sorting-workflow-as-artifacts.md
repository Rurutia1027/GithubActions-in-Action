# Sorting Workflow Data As Artifacts 

### What's "workflow data"?

In GitHub Actions, **workflow data simply means any files, logs, or outputs your job produces**. 
Examples: 
- A .jar file created by Maven (backend/target/app.jar)
- A compiled frontend build directory (frontend/build/)
- Test reports, screenshots, coverage reports
- Logs or CSV exports from a process
- Even packaged Docker iamges (metadata, not the image itself -- large images go to a registry)

Basically, it's the tangible output of a job. 

### What does "sorting workflow data as artifacts" mean? 
It means taking those outptu files from a job and sorting them somewhere safe (in GitHub's artifact storage) so:
- We can download them later from the Actions UI (for debugging or sharing).
- Other jobs in the same workflow can use them as input (by downloading them).
- We keep a clean separation between build, test, deploy steps -- no need to rebuild in later jobs. 


The sorting part just means organizing our output into meaningful named packages like
- build-artifacts (frontend + backend compiled output)
- test-results (JUnit + coverage reports)

### How Artifacts work vs. cache ? 
- **Artifacts**= stored for the duration you configure (default 90 days), meant for passing data between jobs or downloading later. 
- **Cache**= stored between different workflow runs to speed things up (e.g., caching Maven ~/.m2 dependencies).

So: 
- Artifacts: persistent files from this workflow run, retrievable by later jobs. 
- Cache: reusable files between runs, not meant for job-to-job passing. 

### GitHub Action YAML File Supports Upload/Download Workflow Data to Artifacts 

```yml 
name: GitHub Action Workflow with Multiple Jobs - With Artifacts

on: 
  push:
    branches:
      - main

jobs:
  build_job_1:
    name: Build Frontend & Backend 
    runs-on: ubuntu-latest 
    steps:
      - name: Checkout repository 
        uses: actions/checkout@v4

      # Backend Maven Build 
      - name: Set up Java 
        uses: actions/setup-java@v4 
        with:
          java-version: '17'
          distribution: 'temurin'
        
      - name: Build Backend 
        run: |
          cd backend
          mvn clean install -DskipTests 
          mkdir -p ../artifacts/backend
          cp target/*.jar ../artifacts/backend/

      # Frontend Build (based on React)
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
    
      - name: Build Frontend 
        run: |
          cd frontend
          npm ci 
          npm run build
          mkdir -p ../artifacts/frontend
          cp -r build/* ../artifacts/frontend/

      # Save build artifacts
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-artifacts
          path: artifacts/

  test_job_2:
    name: Run Automated Tests 
    runs-on: ubuntu-latest 
    needs: build_job_1 
    steps:
      - name: Checkout repository 
        uses: actions/checkout@v4

      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-artifacts
          path: artifacts/

      - name: Set up Java 
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Run Backend Tests
        run: |
          cd backend 
          mvn test
          mkdir -p ../test-results/backend
          cp target/surefire-reports/* ../test-results/backend/

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Run Frontend Tests
        run: |
          cd frontend 
          npm ci
          npm test -- --watchAll=false
          mkdir -p ../test-results/frontend
          cp -r coverage/* ../test-results/frontend/

      # Save test results
      - name: Upload test results
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: test-results/

  deploy_job_3: 
    name: Deploy Application 
    runs-on: ubuntu-latest 
    needs: test_job_2
    steps:
      - name: Checkout repository 
        uses: actions/checkout@v4

      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-artifacts
          path: artifacts/

      - name: Download test results
        uses: actions/download-artifact@v4
        with:
          name: test-results
          path: test-results/

      - name: Deploy to Server (Example using SCP)
        run: |
          echo "Deploying application using artifacts..."
          # Example:
          # scp artifacts/backend/*.jar user@server:/deploy/
          # scp -r artifacts/frontend/* user@server:/var/www/html/
```

