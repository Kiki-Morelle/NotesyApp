# Notesy Deployment

## Overview

Notesy is a Django application with a TypeScript frontend. The project was containerized with Docker, tested through GitHub Actions, published to Amazon ECR, and deployed to Amazon ECS/Fargate.

## Milestone 1 — Containerization

The application uses a multi-stage Dockerfile.

The frontend is built with Node.js and the final application image uses Python 3.12.

Docker Compose provides:

- Django application
- PostgreSQL 16
- Automatic migrations
- Automatic demo data seeding

Run locally with:

```bash
docker compose up --build

The application is available at:

http://localhost:8000

Demo account:

Username: demo
Password: demo

The Docker container runs as a non-root user.

Milestone 2 — CI

GitHub Actions validates the application with:

Gitleaks secret scanning
Frontend typechecking
Frontend build
Django system checks
PostgreSQL service
Django tests
SonarQube analysis
Docker image build validation

A failing test causes the workflow to fail.

Milestone 3 — Image Publishing

When changes are pushed to main, GitHub Actions authenticates to AWS using GitHub OIDC and publishes the Notesy Docker image to Amazon ECR.

ECR repository:

notesy

Region:

us-east-1

Images are tagged with:

<commit-sha>
latest

The commit SHA identifies the exact version of the image.

JFrog

The project also contains a JFrog publishing step for the devops-project Generic repository.

The Docker image is exported as a TAR archive using docker save:

notesy-<commit-sha>.tar

The JFrog credentials are stored as GitHub Actions secrets:

JFROG_URL
JFROG_USERNAME
JFROG_ACCESS_TOKEN
Milestone 4 — ECS/Fargate Deployment

Notesy is deployed to Amazon ECS using AWS Fargate.

ECS cluster:

notesy-cluster

ECS service:

notesy-service

Task definition:

notesy

Region:

us-east-1

The application listens on port 8000.

The ECS deployment process is:

GitHub Actions authenticates to AWS using OIDC.
The Docker image is built.
The image is pushed to ECR using the commit SHA.
The current ECS task definition is retrieved.
The container image is updated to the new commit SHA.
A new task definition revision is registered.
The ECS service is updated.
ECS starts the new task.
GitHub Actions waits for the service to become stable.
The deployment is verified.
Database

The production database runs on Amazon RDS PostgreSQL 16.

Database:

notesy

The database connection string is stored in AWS Secrets Manager:

notesy/database-url

Database credentials are not stored in the Git repository.

Tradeoffs
ECS/Fargate instead of Kubernetes

The assignment's Kubernetes deployment was a stretch goal. ECS/Fargate was used for the deployment because it provides managed container orchestration without requiring Kubernetes cluster management.

RDS instead of PostgreSQL in ECS

PostgreSQL runs on Amazon RDS instead of inside an ECS container. This separates the database lifecycle from the application lifecycle and provides a managed database service.

Secrets Manager

Production database credentials are stored in AWS Secrets Manager instead of being committed to GitHub or included in the Docker image.

Registry Synchronization

The Git commit SHA is used as the immutable version identifier.

If publishing succeeds in one registry but fails in the other, the failed publishing step is reported by GitHub Actions. The failed registry can then be corrected and the workflow rerun.

The commit SHA allows the exact version requiring synchronization to be identified.

Rollback

To roll back a deployment:

Identify the previous working commit SHA.
Use the corresponding ECR image.
Update the ECS task definition to use that image.
Register a new task definition revision.
Update the ECS service.
Wait for ECS to stabilize.
Verify the application.

Example:

977098999802.dkr.ecr.us-east-1.amazonaws.com/notesy:<previous-commit-sha>

Using the commit SHA avoids relying on the moving latest tag.

Security

The deployment uses:

GitHub Actions OIDC
AWS IAM roles
AWS Secrets Manager
Amazon ECR
Non-root Docker containers
Gitleaks
SonarQube
AWS security groups

No production credentials are committed to the repository.

Monitoring

ECS application logs are sent to Amazon CloudWatch.

Log group:

ecs-notesy

The ECS service can be monitored from the AWS ECS console or with the AWS CLI.