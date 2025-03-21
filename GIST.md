# CI/CD with GitHub Actions for Cypress Testing and Render Deployment

## This tutorial demonstrates how to set up Continuous Integration (CI) and Continuous Deployment (CD) using GitHub Actions to:
* Run Cypress tests when a Pull Request (PR) is made to the develop branch.
* Deploy the application to Render when develop is merged into main.

## Prerequisites
Before starting, ensure you have:
* ✅ A GitHub repository with both a main and develop branch.
* ✅ A Render account for deployment.
* ✅ The application code(in our case cloned from the starter files.)
* ✅ Cypress installed for component testing.

## Step 1: Set Up the Repository
1. Upload the starter code to a new GitHub repository.
2. Create a develop branch from main:

   * git checkout -b develop
   * git push origin develop

3. Ensure that all feature branches are merged into develop, and only develop merges into main.

## Step 2: Create a GitHub Actions Workflow for Cypress Testing

1. Navigate to .github/workflows/ in your repo.
2. Create a new file: checking_tests.yml
3. Add the following YAML configuration:

File Path: .github/workflows/checking_tests.yml
File Content:
```github-actions-workflow
name: Checking Tests

on:
  pull_request:
    branches:
      - develop
      - staging

jobs:
  test:
    name: Checking Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 21.x

      - name: Install dependencies
        run: npm install

      - name: Build App
        run: npm run build

      - name: Run Cypress tests
        run: npm run test-component
```
### What This Does:
* Runs when a PR is made to develop
* Checks out the repository
* Installs dependencies
* Runs Cypress tests

## Step 3: Set Up Deployment to Render

We need a CD workflow that deploys to Render when develop merges into main.

1. Set Up Your Render Application
	1.	Deploy the app on Render and connect it to your GitHub repo.
	2.	Disable Auto-Deploy in Render settings.
	3.	Copy the Deploy Hook URL from Render.

2. Store the Deploy Hook as a GitHub Secret
	1.	In GitHub, go to Settings > Secrets and variables > Actions.
	2.	Click New repository secret.
	3.	Add:
	    * Name: RENDER_DEPLOY_HOOK_URL
	    * Value: Paste the Deploy Hook URL

3. Create the Deployment Workflow
	1.  Navigate to .github/workflows/
	2.	Create a new file: deploy_to_render.yml
	3.	Add the following YAML configuration:

    File Path: .github/workflows/deploy_to_render.yml
File Content:
```github-actions-workflow
name: Deploy To Render

on:
  push:
    branches: [main]
  pull_request:
    branches: 
      - main

jobs:
  ci:
    name: Deploy To Render
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Test
        run: |
          npm install
          npm run build
          npm run test-component

      - name: Deploy
        if: github.ref == 'refs/heads/main'
        env:
          deploy_url: ${{ secrets.RENDER_DEPLOY_HOOK_URL }}
        run: |
          curl "$deploy_url"
```

### What This Does:
*  Runs when develop is merged into main
*  Triggers a manual deploy on Render using the Deploy Hook

## Step 4: Validate the Setup

### Develop branch (Cypress) workflow

1. Create a new feature branch out of the develop branch:
git checkout -b feature

2. Make changes and push to the branch
git add -A
git commit -m "your message"
git push origin feature

3. Open a pull request from feature to develop

4. Check GitHub Actions to see Cypress tests running and completing

### Main branch (deployment) workflow
1. Create a pull request from develop to main

2. Check GitHub Actions to make sure the Cypress tests ran and the deployment was successfully completed

3. Visit your Render URL to confirm the deployment