# Automated Docker Container Deployment Using GitHub Actions and Docker Hub

> **Subject:** DevOps Lab CA Mini Project  
> **Project type:** College mini project (group of up to 3 students)  
> **Core idea:** Automatically test an application, build its Docker image, and publish the image to Docker Hub whenever code is pushed to GitHub.

## 1. Project Overview

Hello

This project demonstrates a simple CI/CD pipeline for a Node.js web application.

Instead of manually testing the code and building/publishing a Docker image after every change, **GitHub Actions automates the process**. When code is pushed to the `main` branch, the workflow:

1. Checks out the repository.
2. Sets up Node.js.
3. Installs project dependencies.
4. Runs automated tests.
5. Builds a Docker image.
6. Logs in to Docker Hub using GitHub repository secrets.
7. Pushes the image to Docker Hub with two tags: `latest` and the commit SHA.

The published image can then be pulled and run as a container. The current workflow **publishes the image**; it does not automatically connect to a remote server to deploy it. Running the image locally is part of the demonstration.

## 2. Why This Project?

Manual deployment steps can be repetitive and easy to forget. This project demonstrates how DevOps practices make the process more consistent:

- **Automation:** GitHub Actions runs the workflow after a push.
- **Testing:** tests run before the image is published.
- **Containerization:** Docker packages the app and its runtime environment.
- **Image registry:** Docker Hub stores the built image so it can be pulled elsewhere.
- **Traceability:** the commit-SHA tag identifies the source revision used to build an image.

## 3. Technologies Used

| Technology | Purpose |
|---|---|
| Node.js | JavaScript runtime |
| Express.js | Lightweight web server |
| Jest | Automated testing |
| Supertest | HTTP endpoint testing |
| Docker | Packages the application into an image and runs it as a container |
| Git | Version control |
| GitHub | Hosts the source-code repository |
| GitHub Actions | Automates testing, image building, and publishing |
| Docker Hub | Stores and distributes Docker images |

This project intentionally keeps the scope focused on Docker, GitHub Actions, and Docker Hub. Kubernetes and Jenkins are not required.

## 4. How the Pipeline Works

```text
Developer edits application
          |
          v
      git push
          |
          v
  GitHub repository (main)
          |
          v
 GitHub Actions workflow
          |
          v
 Install dependencies
          |
          v
    Run automated tests
          |
     tests pass?
          |
          v
    Build Docker image
          |
          v
 Authenticate with Docker Hub
          |
          v
 Push :latest and :<commit-sha>
          |
          v
 Pull and run the image as a container
```

**Important:** if the tests fail, the workflow stops before the Docker image publishing steps. The workflow is triggered by pushes to `main` and can also be started manually using `workflow_dispatch`.

## 5. Application Features

The Express application currently provides two GET endpoints:

| Endpoint | Expected response |
|---|---|
| `/` | JSON message: `DevOps Lab CA application is running!` and version `1.0.0` |
| `/health` | JSON status: `healthy` |

The automated tests in `test/server.test.js` check that both endpoints return HTTP `200` and the expected response values.

## 6. Repository Structure

```text
DevopsLabCA-main/
├── .github/
│   └── workflows/
│       └── docker.yml       # CI/CD workflow
├── src/
│   └── server.js            # Express application
├── test/
│   └── server.test.js       # Automated tests
├── .dockerignore             # Files excluded from Docker build context
├── .gitignore                # Files Git should ignore
├── Dockerfile                # Instructions to build the image
├── package.json              # Dependencies and npm scripts
├── package-lock.json         # Locked dependency versions
└── README.md                 # Project documentation
```

### What the important files do

- **`src/server.js`** — creates the Express server and defines `/` and `/health`.
- **`test/server.test.js`** — verifies the endpoints using Jest and Supertest.
- **`Dockerfile`** — uses `node:24-alpine`, installs production dependencies, copies the app source, and configures port `3000`.
- **`.github/workflows/docker.yml`** — automates dependency installation, testing, Docker Hub login, image build, and image push.
- **`.dockerignore`** — keeps files such as `node_modules`, tests, Git metadata, and workflow files out of the Docker build context.
- **`package.json` / `package-lock.json`** — define the npm scripts and dependencies.

## 7. Run the Application Locally

### Prerequisites

Install:

- Node.js 24 (or a compatible Node.js version)
- npm
- Git
- Docker Desktop (or Docker Engine)
- A GitHub account
- A Docker Hub account

### Install dependencies

From the project root:

```bash
npm ci
```

### Run the tests

```bash
npm test
```

### Start the application

```bash
npm start
```

The server listens on port `3000` by default. Open these URLs in a browser, or test them using curl:

```bash
curl http://localhost:3000/
curl http://localhost:3000/health
```

Expected JSON from `/`:

```json
{
  "message": "DevOps Lab CA application is running!",
  "version": "1.0.0"
}
```

Expected JSON from `/health`:

```json
{
  "status": "healthy"
}
```

To stop the local server, press `Ctrl+C` in its terminal.

## 8. Build and Run the Docker Image Locally

Run these commands from the directory containing the `Dockerfile`.

### Build the image

```bash
docker build -t devops-lab-ca:local .
```

### Run a container

```bash
docker run --name devops-lab-ca-container -p 3000:3000 -d devops-lab-ca:local
```

Visit `http://localhost:3000/` and `http://localhost:3000/health`.

### Useful Docker commands

```bash
docker ps
docker logs devops-lab-ca-container
docker stop devops-lab-ca-container
docker rm devops-lab-ca-container
```

If port `3000` is already occupied, map another host port, for example:

```bash
docker run --name devops-lab-ca-container -p 3001:3000 -d devops-lab-ca:local
```

Then visit `http://localhost:3001/`.

## 9. GitHub Actions and Docker Hub Setup

### Step 1: Push the project to GitHub

Create a GitHub repository and push the project files. Ensure the default branch is `main`, because the workflow is configured to run on pushes to `main`.

Typical commands for a newly initialized local repository:

```bash
git init
git add .
git commit -m "Initial project setup"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
git push -u origin main
```

If Git is already initialized or a remote is already configured, do not repeat the relevant setup commands. Replace the example repository URL with your actual GitHub repository URL.

### Step 2: Create a Docker Hub repository

1. Sign in to Docker Hub.
2. Create a repository named **`devops-lab-ca`**.
3. Choose whether it should be public or private, according to your group/assessment needs.

The workflow publishes image tags in this format:

```text
YOUR-DOCKERHUB-USERNAME/devops-lab-ca:latest
YOUR-DOCKERHUB-USERNAME/devops-lab-ca:<commit-sha>
```

### Step 3: Create a Docker Hub access token

Create an access token in Docker Hub with the permissions required to push to the repository. Use the token in GitHub Actions rather than placing a Docker Hub password or token in source code.

### Step 4: Add GitHub Actions secrets

In the GitHub repository, open **Settings → Secrets and variables → Actions** and add these repository secrets:

| Secret name | Value |
|---|---|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | The Docker Hub access token |

Secret names must match the names used in `.github/workflows/docker.yml`. Never commit tokens or passwords to Git.

### Step 5: Trigger and inspect the workflow

Push a commit to `main`, or open **Actions** in GitHub and manually run **Docker CI/CD** using `workflow_dispatch`.

Inspect the job steps and confirm that:

1. Dependencies install successfully.
2. Tests pass.
3. Docker image build succeeds.
4. Docker Hub login succeeds.
5. Both image tags are pushed successfully.

If the workflow fails, open the failed run and read the first failing step and its logs.

## 10. Pull and Run the Published Image

After a successful workflow run, replace `YOUR-DOCKERHUB-USERNAME` with the username configured in the GitHub secret:

```bash
docker pull YOUR-DOCKERHUB-USERNAME/devops-lab-ca:latest
docker run --name devops-lab-ca-container -p 3000:3000 -d YOUR-DOCKERHUB-USERNAME/devops-lab-ca:latest
```

Visit `http://localhost:3000/` and `http://localhost:3000/health`.

To run a specific build, use its commit-SHA tag instead of `latest`.

## 11. Demonstration Plan

For the lab demonstration, follow this sequence:

1. Explain the project objective and the CI/CD pipeline.
2. Run the application locally and show `/` and `/health`.
3. Show the Dockerfile and explain the main instructions.
4. Show the test file and run `npm test`.
5. Show `.github/workflows/docker.yml`.
6. Explain the two GitHub Actions secrets without revealing their values.
7. Push a change to `main` and show the workflow run in the GitHub Actions tab.
8. Show that tests pass and the image is built and pushed to Docker Hub.
9. Pull and run the published image as a container.
10. Make a small visible change to the application response, update the test if needed, and push again.
11. Show the new workflow run and updated image tags.
12. Run the updated image and demonstrate the changed response.

This demonstrates the full path from code change to tested, published container image.

## 12. Suggested Work Split for a Group of 3

Adjust this split to match each member's contribution. Everyone should understand the complete workflow, not only their assigned part.

| Work area | Suggested responsibility | What the member should be ready to explain |
|---|---|---|
| Application and tests | Express endpoints, npm scripts, Jest/Supertest tests | Routes, HTTP status codes, test purpose |
| Docker | Dockerfile, image build, container run, useful Docker commands | Image vs container, `FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE`, `CMD`, port mapping |
| CI/CD and registry | GitHub Actions workflow, secrets, Docker Hub tags and push | Trigger, job steps, secrets, tags, how the pipeline stops on test failure |

The split is a planning suggestion, not a claim about work already completed by any member.

## 13. Key Concepts for Viva

- **DevOps:** practices that bring development and operations closer together through collaboration, automation, and feedback.
- **CI (Continuous Integration):** automatically validating changes, for example by installing dependencies and running tests after a push.
- **CD (Continuous Delivery/Deployment):** automating later delivery steps. In this project, the workflow automatically publishes the image; starting a container is currently a separate step.
- **Docker image:** a packaged template containing the application and the dependencies needed to run it.
- **Container:** a running instance of an image.
- **Dockerfile:** instructions Docker uses to build an image.
- **Docker Hub:** a registry where Docker images are stored and shared.
- **GitHub Actions:** GitHub's automation platform that runs workflow jobs in response to events or manual triggers.
- **GitHub Actions secret:** a protected value used by a workflow, such as a registry token.
- **Image tag:** a label identifying an image version. `latest` is a conventional tag, not a guarantee that an image is the newest one; the commit-SHA tag helps identify a particular build.
- **Port mapping (`-p 3000:3000`):** maps port `3000` on the host to port `3000` inside the container.
- **Why test before pushing?** It prevents the workflow from publishing an image when the automated tests fail.
- **Why use a commit-SHA tag?** It makes a published build easier to associate with the source commit that triggered it.

## 14. Current Scope and Limitations

The repository currently implements a small Express app, two endpoint tests, a Dockerfile, and a GitHub Actions workflow that tests, builds, and pushes the image.

The current workflow does **not**:
- deploy automatically to a remote cloud server;
- use Kubernetes or Jenkins;
- include a database or frontend framework;
- perform advanced security scanning or production monitoring.

These are outside the current mini-project scope. Keep the demonstration focused and reliable.

## 15. Troubleshooting Checklist

- **Workflow did not start:** confirm the push was to `main`, or use `workflow_dispatch`.
- **Docker Hub login failed:** verify `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` are configured correctly and the token can push to the repository.
- **Image push failed:** confirm the Docker Hub repository exists and the account has permission to push to it.
- **Tests failed:** run `npm ci` followed by `npm test` locally and inspect the failing assertion.
- **Port already in use:** map a different host port, such as `-p 3001:3000`.
- **Container exits:** inspect `docker logs devops-lab-ca-container` and check the Docker run command.
- **Changes are not visible:** confirm the latest workflow completed successfully, pull the intended image tag again, and recreate the container from that image.

---

**Project title:** Automated Docker Container Deployment Using GitHub Actions and Docker Hub  
**Primary learning outcome:** automate application testing and Docker image publishing through a GitHub Actions CI/CD workflow.
