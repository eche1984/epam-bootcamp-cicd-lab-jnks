# CI/CD Pipeline with Jenkins — Security & Quality Gates

A multi-stage Jenkins pipeline (Pipeline as Code) built during the EPAM DevOps bootcamp, 
demonstrating a Node.js application build process with integrated security and quality gates: 
Dockerfile linting, container vulnerability scanning, and branch-based multi-environment deployment.

## Stack

- **CI/CD:** Jenkins (Declarative Pipeline, Shared Library)
- **Containerization:** Docker (custom Jenkins agent image + application image)
- **Linting:** Hadolint (Dockerfile best practices)
- **Security:** Trivy (container vulnerability scanning)
- **App:** Node.js
- **Registry:** Docker Hub

## Pipeline Flow

1. **Checkout** — pulls source code from SCM
2. **Set environment variables** — branch-aware configuration (`main` vs `dev`), assigning a 
   distinct Docker image name, host port, and container name per branch
3. **Build** — installs dependencies via `npm install`
4. **Test** — runs the test suite (`npm test`)
5. **Dockerfile Linting** — validates the Dockerfile against best practices using Hadolint, 
   failing the build on errors
6. **Build Docker Image** — builds the application image, tagged per branch
7. **Security Scan** — scans the built image for critical vulnerabilities using Trivy
8. **Push to Docker Hub** — publishes the image using a shared library function (`dockerUtils`)
9. **Trigger downstream** — kicks off the corresponding deploy job (`Deploy_to_main` / 
   `Deploy_to_dev`) based on branch
10. **Post** — workspace cleanup on every run

## Custom Jenkins Agent

The pipeline runs on a custom Docker agent (`Dockerfile.agent`) built on Ubuntu 20.04, 
pre-installed with Node.js, Docker CLI, Hadolint, and Trivy — enabling linting, image builds, 
and security scanning without relying on the host environment.

## Files

- `Jenkinsfile` — pipeline definition (declarative syntax, uses shared library)
- `Dockerfile` — application image definition (Node.js app)
- `Dockerfile.agent` — custom Jenkins agent image with build/lint/scan tooling

## Context

Built as a practical exercise during the EPAM DevOps bootcamp, focused on implementing CI/CD 
quality and security gates (linting + vulnerability scanning) alongside a branch-based 
multi-environment deployment strategy — practices commonly expected in production-grade 
Jenkins pipelines.

## Related project

This pipeline consumes the [`epam-bootcamp-jnkns-shared-library`](https://github.com/eche1984/epam-bootcamp-jnkns-shared-library) 
for its Docker Hub publishing step (`dockerUtils` function), centralizing image tagging and 
authenticated registry push logic outside the main Jenkinsfile.
