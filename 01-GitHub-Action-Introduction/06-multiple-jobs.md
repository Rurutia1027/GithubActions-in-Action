# Workflow with Multiple Jobs 

```yml 
name: GitHub Action Workflow with Multiple Jobs 

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

      # Frontend Build (based on React)
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
    
      - name: Build Frontend 
        run: |
          cd frontend
          npm ci 
          npm run bild 
  
  test_job_2:
    name: Run Automated Tests 
    runs-on: ubuntu-latest 
    needs: build_job_1 
    steps:
      - name: Checkout repository 
        uses: actions/checkout@v4

      - name: Set up Java 
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Run Backend Tests
        run: |
          cd backend 
          mvn test 
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Run Frontend Tests
        run: |
          cd frontend 
          npm ci
          npm test -- --watchAll=false 
  
  deploy_job_3: 
    name: Deploy Application 
    runs-on: ubuntu-latest 
    needs: test_job_2
    steps:
      - name: Checkout repository 
        uses: actions/checkout@v4

      - name: Deploy to Server (Example using SCP)
        run: |
          echo "Deploying application ..."

```