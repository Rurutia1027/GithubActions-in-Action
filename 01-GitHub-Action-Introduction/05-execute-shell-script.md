# Execute Shell Script in Action 

- Suppose we have a GitHub Action YAML file 

```yml 
name: GitHub Action Workflow of Executing Shell Script 

on: push 

jobs:
  first_job:
    runs-on: ubuntu-latest
    steps: 
    - name: Checkout Repo 
      uses: actions/checkout@v4

    - name: List Repo Files 
      run: ls -ltra 

    - name: Executing Shell Script 
      run: ./my-script.sh
```

- And a Shell script .sh file 

```sh
!#/bin/sh 

echo "Shell Script Executing"
exit; 
```