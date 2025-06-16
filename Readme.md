# k8s-api-app Helm Chart

This is a Helm chart for deploying the `api-app`, a simple Express-based API, to a Kubernetes cluster. This chart is part of a DevOps test project focused on demonstrating containerization, CI/CD, infrastructure as code, and Kubernetes orchestration.

## Features

- Deploys the `api-app` as a Kubernetes Deployment
- Exposes the application via a Kubernetes Service (ClusterIP/NodePort/LoadBalancer)
- Configurable via `values.yaml`
- Supports Docker image versioning and resource customization

## Requirements

- Kubernetes 1.19+
- Helm 3.x
- Docker image published to DockerHub (e.g. `radeczu/apitestapp`)