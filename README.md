# CI/CD Pipeline Demo

A minimal demonstration of a **CI/CD pipeline** using **GitHub Actions**.

---

## What is CI/CD?

- **CI (Continuous Integration):** Automatically build, test, and validate code every time changes are pushed to a repository. Catches bugs early.
- **CD (Continuous Delivery/Deployment):** Automatically deploy validated code to staging or production environments.

CI/CD pipelines run on **runners** — virtual machines or containers that execute your jobs whenever a trigger event (like a `push`) occurs.

---

## GitHub Actions Basics

GitHub Actions is GitHub's built-in CI/CD platform. You define workflows as YAML files inside `.github/workflows/`.

### Key Concepts

| Concept | Description |
|---|---|
| **Workflow** | A single automated process defined in a YAML file |
| **Event** | What triggers the workflow (push, pull_request, schedule, etc.) |
| **Job** | A set of steps that run on the same runner |
| **Step** | An individual task (run a command or use an action) |
| **Action** | A reusable unit of code (community or custom) |
| **Runner** | The virtual machine that executes the workflow (`ubuntu-latest`, `windows-latest`, `macos-latest`) |

---

## Example: The Pipeline in This Repo

```yaml
# .github/workflows/main.yml

name: CICD Pipeline in GitHub Actions

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v5

      - name: Initialize the Repo
        run: |
          echo "Hello World from the GitHub Actions"
          uname -a

      - name: Check the Docker installation
        run: |
          docker
          docker images

      - name: Run Docker Container
        run: |
          docker run hello-world
```

### What it does

| Step | Purpose |
|---|---|
| `actions/checkout@v5` | Checks out your repository code so later steps can access it |
| **Initialize the Repo** | Prints a greeting and system info (`uname -a`) to verify the runner environment |
| **Check Docker installation** | Verifies Docker is available and lists existing images |
| **Run Docker Container** | Runs `hello-world` to confirm Docker can spin up containers |

### How to see it in action

1. **Push something** to the `main` branch of this repo.
2. Go to the **Actions** tab of the GitHub repository.
3. Click the running (or completed) workflow to see live logs for each step.

---

## Common CI/CD Patterns

### 1. Lint, Test, Build (Language-agnostic pseudo-workflow)

```yaml
name: Lint-Test-Build

on: [push, pull_request]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test

      - name: Build
        run: npm run build
```

### 2. Docker Build & Push

```yaml
name: Docker Build & Push

on:
  push:
    branches: [main]

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: myusername/myapp:latest
```

### 3. Multi-job Workflow (Build → Test → Deploy)

```yaml
name: Multi-stage

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - run: npm ci && npm test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-output
      - run: echo "Deploying to production..."
```

---

## Useful Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax Reference](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Marketplace — Community Actions](https://github.com/marketplace?type=actions)
