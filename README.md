# Github-actions

GitHub Actions have five important components:

1) Workflow — like a pipeline in Jenkins.

2) Events — triggers on which the workflow is run (e.g. Push, Pull Request, Scheduled).

3) Jobs — stages like Build, Test, Static Code Analysis.

4) Actions — reusable tasks inside jobs.

5) Runners — the platform / operating system that will execute the jobs.

Sample GitHub Flow:

.gitlab/workflow/.test_gha.yaml — GitHub Flow is to be created under Actions in GitHub, inside a repository.

After your repo is created, go to the Actions tab. There, you can create a workflow.

You can run the workflow manually as well. Also, you can disable it if you don’t want it triggered by any event.

Sampe workflow file structure:
```
name: demo_workflow        # “demo_workflow”: name of the workflow

on:                        # event on which the workflow is triggered.

  push:                    # triggers on push
    branches: [ "main" ]

  pull_request:            # triggers on pull request
    branches: [ "main" ]

  workflow_dispatch:       # allows manual run via the “Run workflow” button

jobs:                      # job is set of tasks that workflow executes

  build:                     # name of a job (it is unique always)
    runs-on: ubuntu-latest    # runner: Ubuntu environment

  steps:                      # steps are individual tasks under jobs
      - uses: actions/checkout@v4
      - name: Run a one-line script   # name of the step
        run: echo "Hello, test change!"
      - name: Run a multi-line script
        run: |
          echo "Add other actions to build,"
          echo "test, and deploy your project."
```
In the above code, 'uses: actions/checkout@v4' : It uses a pre-built GitHub Action that clones your repository into the workspace on the runner machine. You always need your code to test, build, and do static code analysis. Hence, you almost every time use this line in the workflow.

# Variables in github actions: 

i) Environment variable-
env: This keyword is used to define variables in GitHub Actions. You can define variables scoped for:

1) Entire workflow

2) Contents of a job within a workflow

3) Specific step within a job

So, in short, env variables are defined within a workflow. They are in key-value pairs, where key is used to call the variable (with a dollar sign). For example, $cloud: google_cloud.

```
name: workflow_variable
on:
  workflow_dispatch
env:
  cloud: google_cloud    # Variable one (workflow level)

jobs:
  job_one:
    runs-on: ubuntu-latest
    environment:
      greet: hello        # Variable two (job level)
    steps:
      - name: "greet the viewer"
        run: echo "$greet $first_name, welcome to $cloud!"
        env:
          first_name: Alex  # Variable three (step level)
```

Why this matters & supported behaviour (based on official docs):

Environment variables (env) can be defined at workflow, job, or step level. The scope of the variable depends on where it is defined: workflow-level variables are visible everywhere, job-level only inside that job, step-level only inside that one step. 
Default and custom variables, plus secrets, are supported.

ii) Configuration variable-
Now, if we want to declare a variable which can be used in multiple workflows in the repository, then we use configuration variables.

We define configuration variables in the settings of a repository, not inside the workflow’s YAML file. We use them in workflows by syntax ${{ vars.variableName }}.

To define a configuration variable (variableName)as : Go to Settings in the repository -> Then Secrets and variables → Actions -> Click New repository variable, define the name & value, and save it

Now you can run the workflow.

For values related only to a specific workflow, we use workflow variables. But for more common variables, we use configuration variables.

Context variable are preloaded set of information like who triggered the workflow, or repository name, or other metadata. These are available via the context from GitHub metadata.

Sometimes we have many conditions to run the workflow. For example, “if repo name is xyz”, “if job name is xyz”, or “if environment is xyz”, then run certain parts. That metadata (repo, workflow, or job) is accessed using context.

Example code using context variables:

```

name: context_variable_demo
on:
  workflow_dispatch

env:
  CLOUD: google

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Print repository and workflow info
        run: |
          echo "Repository name: ${{ github.repository }}"
          echo "Workflow name: ${{ github.workflow }}"
          echo "Triggered by: ${{ github.actor }}"
```

In the last three lines, we haven’t defined these variables anywhere. They are built-in and pulled from GitHub Actions contexts.

# Manual input and secrets:

>> Input under workflow_dispatch: You can also give manual input when triggering a manual workflow. We define this under inputs in the workflow_dispatch trigger. For syntax, refer to the GitHub Actions documentation or Vishal Bulbule’s repository. Then, when you manually trigger the workflow, you will get a window to enter those values. Inputs under workflow_dispatch allow manual workflows to ask for values. You define them in on.workflow_dispatch.inputs. 

>> Secrets: Now coming to secrets: you cannot pass secrets directly in the workflow file (hard-code them). Hence, you need to create secrets in GitHub. To create secrets, go to Settings → Secrets and variables → Actions. Then click New repository secret. Add the secret name and value and save it. 

>> In the workflow code, you use the secret the same way as configuration variables: ${{ secrets.secretName }}. When you run the step, the output will show stars (***) instead of the actual secret (masked). Use secrets with syntax ${{ secrets.SECRET_NAME }}. They are masked in logs.

Example of secrets and manual input are as below - (The workflow is this same repo : 'input_manual_trigger')

```
name: workflow_with_inputs_and_secrets
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Which environment to deploy to'
        required: true
        default: 'staging'
      run_tests:
        description: 'Run tests before deploy'
        required: false
        type: boolean
        default: false

jobs:
  job_one:
    runs-on: ubuntu-latest   # or self-hosted tag if using your own runner
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Greet with inputs
        run: echo "Deploying to ${{ github.event.inputs.environment }}"

      - name: Use a secret
        run: echo "Secret is ${{ secrets.test }}"  # will be masked
```
Output of manul input is as below (we get an option to mention environment)

<img width="1735" height="742" alt="Screenshot 2025-09-19 at 5 29 15 PM" src="https://github.com/user-attachments/assets/8505620f-997a-4a22-b7f1-ce51e56df972" />



# Runners:

Runners are machines that execute the jobs defined in GitHub Actions workflows. A workflow is a set of jobs; each job is a set of steps (commands). But where are these steps executed? On runners.

There are two types of runners:

1) GitHub-hosted runners: Virtual machines provided by GitHub. They come preconfigured (OS, tools installed, etc.). GitHub manages their maintenance.

2) Self-hosted runners: Machines you provide and manage (e.g. your EC2 instance). You install the runner software on them. You ensure network, security, uptime, etc.

To use a self-hosted runner:

Create the server in AWS or Azure as you normally do and then connect to them via SSH/terminal.
Ensure SSH (port 22) access if needed.
In your GitHub repository, go to Settings → Actions → Runners. In the Runners settings page, you’ll see commands to register the runner. Copy and execute those commands on the EC2 instance.

After registration, the runner appears in GitHub as active/self-hosted. In the workflow file use runs-on: self-hosted (or tag matching your runner) so GitHub knows to use your self-hosted runner.

When the EC2 instance (runner) is turned off or disconnected, the runner shows offline in GitHub. If you trigger a workflow then, jobs stay in queue. Also, not everyone has permission to register runners: depends on repository organization permissions. You can disable the self-hosted runner option in settings if needed.

Now configure value as 'self-hosted' for 'on' value in workflow as below-
```
runs-on: self-hosted

```

You need to implement the below commands to configure any server as runner (as mentioned above these commands are given in runner tab in Github). Also it is litening to jobs and running it. Once our workflow runs we can exit the runner. If after exiting runner if we go run the workflow again then it goes into queued state

<img width="1792" height="983" alt="Screenshot 2025-09-19 at 5 27 08 PM" src="https://github.com/user-attachments/assets/0f957625-05f8-4a5c-b28b-905f293c04a9" />

---

Notes of github actions workflow:

Think of a workflow as a pipeline of jobs. Each job is a box that runs on a runner (host). Jobs can run in parallel or depend on others (needs). Build the pipeline so:

- Cheap checks (lint/unit tests) run first and in parallel.

- Expensive steps (integration tests, security scans, image builds) run after cheap steps pass.

- Artifacts (test results, coverage reports, built image metadata) are passed between jobs via uploads/downloads or a registry.

- Final step is deployment, gated by environment protection or manual approval in production.

**Key concepts to use:**

jobs and needs (dependencies)

runs-on (ubuntu-latest vs self-hosted)

steps inside jobs (checkout, setup, run)

caching (actions/cache) for speed

artifacts (actions/upload-artifact) to pass files between jobs

secrets (DOCKERHUB, KUBECONFIG) stored in repo/org secrets

concurrency to prevent overlapping runs

env for job-level env vars

if: for conditional execution (e.g., only on main branch, only on tags)

matrix for testing multiple versions (Node/Java/Python)

**Step-by-step plan you should follow when writing a new pipeline**

Define scope: which branches, PRs, or tags trigger it? (e.g., push to main, pull_request for PRs).

Minimum pipeline: Make a tiny workflow that checks out code and runs tests. Get that green on PRs.

Add lint/static analysis: fast checks that fail the PR early.

Add unit tests and coverage: collect coverage artifacts.

Add security scans (SAST/Dependency): e.g., CodeQL, dependency-audit.

Add build and package (docker build). Use cache for speed.

Publish artifact/image to a registry.

Staging deployment: deploy to k8s cluster for further tests.

Production deployment: gated (manual approval / protected env).

Observability: add logs, notifications (Slack/email) for failures.

Optimize: parallelize jobs, cache dependencies, move heavy jobs to self-hosted runners if needed.

Always iterate. Don’t try to do everything in one commit. Get tests → then build → then deploy.

Secrets & environment you’ll need (store in GitHub repo settings → Secrets)

DOCKERHUB_USERNAME / DOCKERHUB_TOKEN (or registry user/token)

KUBECONFIG (base64-encoded kubeconfig or use cluster credentials via a GitHub Action that authenticates)

COVERAGE_TOKEN (if you upload to codecov/coveralls)

Any cloud provider credentials for login (AWS/GCP/Azure) if you use ECR/GCR/ACR

SLACK_WEBHOOK (optional for notifications)

Never hardcode secrets in your YAML.

**Full sample workflow**

This is a general, practical pipeline for a containerized app (Node/Python/Java — language agnostic). It:

Runs on PRs and pushes to main

Runs lint + unit tests in parallel

Runs CodeQL (security SAST)

Uploads coverage

Builds and pushes Docker image (on main)

Deploys to Kubernetes (on main)

Save as .github/workflows/ci-cd-k8s.yml.

name: CI/CD to Kubernetes

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
  workflow_dispatch: {}

concurrency:
  group: ci-cd-${{ github.ref }}            # github.ref = the branch or tag that triggered the workflow. We dont need to define this github.ref anywhere, github crewtes/picks it up automatically. You only reference it. GitHub replaces ${{ github.ref }} → refs/heads/main at runtime. Stops multiple workflows for the same branch from running at the same time → cancels earlier ones.
  
  cancel-in-progress: true                  # If a new pipeline starts on the same branch, GitHub cancels the older one.

env:                                                               # these are global variables, Defines variables for Docker image name and tag so all jobs use consistent naming → prevents mistakes.
  IMAGE_NAME: ghcr.io/${{ github.repository_owner }}/myapp         # github.repository_owner = your GitHub username or org
  IMAGE_TAG: ${{ github.sha }}                                     # github.sha = commit ID. So your image tag becomes the exact commit hash. So your image built will be: ghcr.io/your-username/myapp:d7e983f8c822ab53e913a2bdd5ab5380

jobs:
  # -------------------------
  # quick checks: lint + unit tests (parallel)
  # -------------------------
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps: 
      - uses: actions/checkout@v4                    
      - name: Setup Node (example)                  
        uses: actions/setup-node@v4                  # uses: means you are calling an existing action — basically someone else’s script packaged for GitHub. There is zero magic. It’s just: download this action, run its code, pass it inputs
        with: node-version: 18                       # with: is how you pass inputs to the action.It’s the equivalent of passing arguments to a function.
      - name: Install deps (cache)                   # There is no universal syntax for with: because it depends on the action you are using. Every action has its own README that defines: Allowed inputs, output variables, examples Use readme for each action before using it.
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
      - name: Install
        run: npm ci
      - name: Lint
        run: npm run lint

  unit-tests:
    name: Unit tests & Coverage
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with: node-version: 18
      - name: Install deps
        run: npm ci
      - name: Run tests with coverage
        run: npm test -- --coverage --coverageDirectory=coverage
      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage

  # -------------------------
  # Static analysis / Security
  # -------------------------
  codeql:
    name: CodeQL Security Scan
    runs-on: ubuntu-latest
    permissions:                          # GitHub Actions workflows run with a token called GITHUB_TOKEN. By default, this token has broad permissions (read/write). This is a security risk. So workflows can override the token’s power using this. “Give this workflow only read access to Actions and repository contents. Nothing more.”
      actions: read      
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: javascript
      - name: Autobuild
        uses: github/codeql-action/autobuild@v2
      - name: Run CodeQL queries
        uses: github/codeql-action/analyze@v2

  # -------------------------
  # Build docker image (only on main) and push to registry
  # -------------------------
  build-and-push:
    name: Build and Push Docker Image
    runs-on: ubuntu-latest
    needs: [unit-tests, codeql]
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      - name: Login to registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          push: true
          tags: ${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      - name: Upload image-tag artifact
        uses: actions/upload-artifact@v4
        with:
          name: image-tag
          path: image-tag.txt
      - name: Save IMAGE TAG
        run: echo "${{ env.IMAGE_TAG }}" > image-tag.txt

  # -------------------------
  # Deploy to Kubernetes (staging or main)
  # -------------------------
  deploy:
    name: Deploy to Kubernetes
    runs-on: ubuntu-latest
    needs: build-and-push
    if: github.ref == 'refs/heads/main'
    environment:
      name: staging                                     # Says: “This job is deploying to the staging environment.”
      url: https://staging.example.com                  # “If this workflow is running on the main branch → treat this job as a staging deployment and attach the staging URL.”
    steps:
      - uses: actions/checkout@v4
      # download the image-tag artifact from previous job (example)
      - name: Download image-tag
        uses: actions/download-artifact@v4
        with:
          name: image-tag
          path: .
      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: 'v1.27.0'
      - name: Configure Kubeconfig
        run: |
          echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig
          export KUBECONFIG=$PWD/kubeconfig
        shell: bash
      - name: Update image in deployment
        run: |
          IMAGE="${{ env.IMAGE_NAME }}:$(cat image-tag.txt)"
          kubectl -n mynamespace set image deployment/myapp myapp=$IMAGE
          kubectl -n mynamespace rollout status deployment/myapp --timeout=120s
      - name: Run smoke tests
        run: ./scripts/smoke-test.sh

  # Optionally: notify on failure (runs always)
  notify:
    name: Notify Slack on Failure
    runs-on: ubuntu-latest
    needs: [build-and-push, deploy]
    if: failure()
    steps:
      - name: Send slack message                    # “Run this job/step ONLY if a previous job/step in the workflow has failed.” failure() is a GitHub Actions built-in function that returns true when anything earlier failed. So this block is part of a failure-handling job.
        run: |
          curl -X POST -H 'Content-type: application/json' --data '{"text":"CI failed for ${{ github.repository }}"}' ${{ secrets.SLACK_WEBHOOK }}


Notes:

Replace npm steps with your language build/test commands.

For Docker registry, I used GitHub Container Registry as example; swap with DockerHub/AWS ECR and use proper login action & secrets.

KUBECONFIG should be stored as base64 of ~/.kube/config or use GitHub cloud provider actions to authenticate (e.g., aws-actions/amazon-eks-login for EKS).

**Why this structure (explain line-by-line thinking)**

lint first: cheap, fails fast.

unit-tests depends on lint: we don’t waste test compute if lint fails.

codeql can run in parallel because it doesn’t need a built image.

build-and-push requires tests + codeql and only runs on main — prevents unintentional image publish for feature branches.

deploy needs build-and-push — prevents deploying until you have a new image.

Artifacts: we pass the image tag via artifact to deployment job to ensure the exact image is deployed.

concurrency: prevents multiple concurrent deployments for same ref.

if: github.ref == 'refs/heads/main' prevents pushing images for PRs.

**Practical tips & best practices (from experience)**

Use actions/cache for language deps to speed builds.

Fail fast: run lint/test early and in parallel.

Use upload-artifact for test reports & coverage so you can view them on the run page.

Set timeouts for long commands using timeout or job-level timeout-minutes.

Use environments in GitHub for protected deployments (require reviewers for production).

Use GITHUB_TOKEN for ghcr login or dedicated registry tokens.

Prefer specific action versions (use @v4 instead of @master).

Test locally with act to iterate faster (note: not all actions work locally).

Use self-hosted runners for heavy builds or to access private infra (e.g., on-prem Kubernetes).

Use matrix to test multiple runtime versions (Node 16/18/20).

Set up branch protection rules so PR cannot be merged until CI passes.

Keep workflows modular — consider reusable workflows once you have multiple repos.

**Exercises to build skill quickly (do in this order)**

Create a minimal workflow: checkout + echo hello.

Add unit tests and run on PRs.

Add caching for dependencies.

Add lint job that runs in parallel.

Add Docker build + push to your personal registry on main.

Add a deploy job that updates a Kubernetes deployment (use a test cluster or k3d).

Add CodeQL and a dependency vulnerability scan.

Add manual approval for production environment deploys.

Convert repeated logic into a reusable workflow and call it from another repo.

Do these one at a time and validate on GitHub Actions UI.

**Debugging workflows — fast checklist**

If an action fails: open the job log, expand the failing step, copy the exact command and run locally or in a runner shell.

Use echo / env to dump variables (be careful not to leak secrets).

If you see authentication errors: confirm secrets and token scopes.

If kubectl fails: check kubeconfig contents and network access from runner.

If builds are slow: inspect cache hit/miss lines in logs.

For flaky tests: record test artifacts and re-run job with retries or isolate flaky tests.

If you want, next I’ll do one of these (pick one — I’ll produce code/config ready to run):

Tailor the sample workflow to your stack (Node/Python/Java + exact commands).

Create a deploy-to-k8s job that uses GitHub OIDC to authenticate to AWS EKS (no kubeconfig secret).

Convert the workflow into reusable workflows (for mono-repo / multi-repo reuse).

Give you a step-by-step lab using k3d on your laptop to test deployments locally.

---

5 Dec 2025

---

#  pull_request_target:
#   types: [opened, ready_for_review]

✔ pull_request_target

Runs in the context of the base repository (the main repo), NOT the PR code.

Meaning:

Workflow runs using main branch code, not PR code

Has full repository permissions

Has access to secrets

This is dangerous if you don’t know what you're doing.

Why does GitHub even allow this?

Because maintainers sometimes need to:

Add labels

Comment on PRs

Run tests with full permissions

Use secrets for deployments (rare, risky)

You never use this for running untrusted code from a fork.
It is a security disaster if misused.

_____

GitHub Actions supports event filters.

Example:

on:
  pull_request:
    types: [opened, synchronize, reopened]


Meaning:

Don’t trigger on all PR events, only the ones listed.

Common PR event types:

opened

synchronize (new commits pushed to PR)

reopened

ready_for_review

closed

assigned

labeled

…many more

Types = sub-events inside pull_request.

Think of pull_request as the category, and types as specific actions inside that category.

-----

✔ pull_request

Runs in the context of the PR branch.

Uses the fork’s code

Permissions are restricted

Safe but limited

This is the default one.

-----

In Github actions the reusable code available publicaly is nithing but 'Github actions'.

----

10 Dec 2025

Script 1:

name: practice_pipeline

'''
on: 
    push:
    pull_request:
        branches: [main]
    workflow_dispatch:

jobs:
    practice_job1:
        runs-on: ubuntu-latest

        steps:
            - uses: actions/checkout@v4
            - name: multi line script
              run: echo Hello! this is line one
            - name: Line 2 of multi line script
              run: |
                echo this is line 2
                echo And this is line 3
'''

Above script is basic.

1) Giving name for the workflow.
2) Giving event on which it should get triggered.
3) And then jobs. There can be multiple steps under 1 job and in such way there can be multiple jobs. Like in above example practice_job1 is the first job and the steps under it are multi line script and Line 2 of multi line script.
4) Runs-on is another important point in GHA. It is the runnner on which the pipeline will be running.
5) And 'uses' is another important keyword which is used to use actions in GHA. Here we are using predefined action of checkout the source code for the repo.

   Doubt: How shell or linux commands like echo are used here?
   And what is run? Is it an action or?
   Pipeline to work on tomorrow.
    
