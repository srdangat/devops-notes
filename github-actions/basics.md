# GitHub Actions

---

## Key Concepts

- **Workflow**: YAML file in `.github/workflows/` that defines automation.  
- **Job**: A unit of work that runs on a **runner** (Ubuntu, Windows, MacOS).  
- **Step**: A single command or action in a job.  
- **Action**: Reusable functionality that can be shared between workflows.  
- **Runner**: The environment that executes jobs (`ubuntu-latest`, `windows-latest`, `macos-latest`).  

---

## Understanding the Anatomy of a Workflow

- **`on:`**  
  - Defines **when the workflow is triggered**.  
  - Listens for events like `push`, `pull_request`, or `workflow_dispatch`.  

- **`jobs:`**  
  - Defines the **jobs that the workflow will execute**.  
  - A workflow can have **one or multiple jobs**.  

- **`runs-on:`**  
  - Specifies the **virtual machine (runner) environment** for the job.  
  - Examples: `ubuntu-latest`, `windows-latest`, `macos-latest`.  

- **`steps:`**  
  - Defines the **sequence of actions** the job will execute.  
  - Steps run **one after another** inside the job.  

- **`uses:`**  
  - Tells GitHub to use a **prebuilt action**.  
  - Example: `actions/checkout` to clone the repository.  

- **`run:`**  
  - Executes **commands directly** on the runner.  

- **`name:` (on a step)**  
  - Gives the step a **human-readable label** in the Actions UI.


## Basic GitHub Actions Workflow Structure

```yaml
# Name of the worflow
name: hello worflow

# Trigger the workflow on every push
on:
  push:
    branches:
      - main

# Define the jobs that will run in this workflow
jobs:
  greet:

    # specify the runner environment
    runs-on: ubuntu-latest
    
    # Steps represents of tasks executed as a part of the job
    steps:
      # Step 1: Checkout the repository 
      - name: Check repository
        uses: actions/checkout@v4
        
      # Step 2: Print greeting
      - name: Print greeting
        run: echo "Hello from GitHub Actions!"

      # Step 3: Print the current date and time
      - name: Print the current date and time
        run: date

      # Step 4: Print branch name
      - name: Print branch name
        run: |
          echo "Triggered by branch: ${{ github.ref_name }}"

      # Step 5 List repo file
      - name: List repo files
        run: ls -la

      # Step 6 Print runner operating system
      - name: Print runner OS
        run: |
          echo "Runner OS: ${{ runner.os}}"
```