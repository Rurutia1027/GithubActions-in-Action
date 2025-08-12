# Multi-Line Commands and Execution Third Party Libraries 

```yml
name: GitHub Action Workflow 

on: push 

jobs:
  first_job:
    runs-on: ubuntu-latest
    steps: 
    - name: Checkout Repo 
      uses: actions/checkout@v4

    - name: List and Read File 
      run: |
        echo "GitHub Action Job"
        ls -ltra
        cat README.md

    - name: Create A Markdown File
      run: |
        touch file.md 
```