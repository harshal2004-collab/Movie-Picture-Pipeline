# Movie Picture Pipeline — Submission

**Repository:** [https://github.com/harshal2004-collab/Movie-Picture-Pipeline](https://github.com/harshal2004-collab/Movie-Picture-Pipeline)

## Live URLs

- **Frontend:** [http://af5bc511c5ef14c31b0e04f27127d1dc-516381283.us-east-1.elb.amazonaws.com/movies](http://af5bc511c5ef14c31b0e04f27127d1dc-516381283.us-east-1.elb.amazonaws.com/movies)

- **Backend API:** [http://a826abd28ecf94e6ab69025937332fa5-555668542.us-east-1.elb.amazonaws.com/movies](http://a826abd28ecf94e6ab69025937332fa5-555668542.us-east-1.elb.amazonaws.com/movies)

The frontend is deployed on Amazon EKS and retrieves the movie list from the backend API using the `REACT_APP_MOVIE_API_URL` environment variable. The backend API is exposed through an AWS LoadBalancer, and the frontend is configured to communicate with this backend endpoint.

## Pipeline Overview

The project implements a complete CI/CD pipeline using GitHub Actions, Docker, Amazon ECR, Amazon EKS, Kubernetes, and Kustomize.

Four GitHub Actions workflows are configured in `.github/workflows/`:

| Workflow | File | Trigger | Jobs |
|---|---|---|---|
| Frontend CI | `frontend-ci.yaml` | Pull request to `main` affecting frontend files + manual | Lint, Test → Build |
| Backend CI | `backend-ci.yaml` | Pull request to `main` affecting backend files + manual | Lint, Test → Build |
| Frontend CD | `frontend-cd.yaml` | Push/merge to `main` affecting frontend files + manual | Lint, Test → Build → Push to ECR → Deploy to EKS |
| Backend CD | `backend-cd.yaml` | Push/merge to `main` affecting backend files + manual | Lint, Test → Build → Push to ECR → Deploy to EKS |

### Frontend CI

The Frontend Continuous Integration workflow:

- Runs when frontend changes are submitted through a pull request to `main`.
- Can also be triggered manually.
- Installs frontend dependencies using `npm ci`.
- Runs the frontend lint checks.
- Runs frontend tests.
- Builds the frontend Docker image after lint and test jobs succeed.

### Backend CI

The Backend Continuous Integration workflow:

- Runs when backend changes are submitted through a pull request to `main`.
- Can also be triggered manually.
- Installs backend dependencies.
- Runs backend lint checks.
- Runs backend tests.
- Builds the backend Docker image after lint and test jobs succeed.

### Frontend CD

The Frontend Continuous Deployment workflow:

- Runs on frontend changes merged/pushed to `main`.
- Runs frontend lint and tests.
- Builds the production Docker image.
- Passes `REACT_APP_MOVIE_API_URL` to the Docker build.
- Authenticates with Amazon ECR using GitHub Actions.
- Pushes the image to the frontend ECR repository using the Git commit SHA as the image tag.
- Configures access to the EKS cluster.
- Updates the Kubernetes frontend deployment using Kustomize.
- Verifies that the frontend deployment becomes ready.

### Backend CD

The Backend Continuous Deployment workflow:

- Runs on backend changes merged/pushed to `main`.
- Runs backend lint and tests.
- Builds the backend Docker image.
- Authenticates with Amazon ECR.
- Pushes the image to the backend ECR repository using the Git commit SHA as the image tag.
- Configures `kubectl` access to Amazon EKS.
- Deploys the backend using Kustomize.
- Verifies the backend deployment status.

## AWS Infrastructure

The application is deployed using the AWS infrastructure created through Terraform.

### EKS

- **EKS Cluster:** `cluster`
- **Kubernetes Version:** `1.34`
- **Region:** `us-east-1`
- **Worker Nodes:** Managed EKS node group
- **Frontend:** Kubernetes Deployment + LoadBalancer Service
- **Backend:** Kubernetes Deployment + LoadBalancer Service

### Amazon ECR

Two ECR repositories are used:

- **Frontend:** `449030632525.dkr.ecr.us-east-1.amazonaws.com/frontend`
- **Backend:** `449030632525.dkr.ecr.us-east-1.amazonaws.com/backend`

Docker images are tagged using the GitHub commit SHA so that every pipeline build produces a traceable image.

## GitHub Actions AWS Authentication

GitHub Actions connects to AWS using an IAM user created for the CI/CD pipeline.

**IAM User:**

`github-action-user`

AWS credentials are stored in GitHub Secrets and are not hardcoded inside the workflow files.

The workflows use:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`
- `EKS_CLUSTER_NAME`

The AWS credentials are supplied to GitHub Actions through encrypted repository secrets.

## Frontend Configuration

The frontend requires the backend API URL during the production Docker build.

The following environment variable is used:

```text
REACT_APP_MOVIE_API_URL