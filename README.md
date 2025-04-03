# Kubernetes Guestbook Application

A simple, scalable web application built with Docker and Kubernetes that allows users to leave messages in a guestbook.

## Project Overview

This project demonstrates several key Kubernetes concepts:
- Building and deploying containerized applications
- Horizontal Pod Autoscaling
- Rolling updates and rollbacks
- Resource management
- Multi-stage Docker builds

## Architecture

The application consists of a web front-end built with Go that allows users to input and submit text messages. Messages are displayed above the input box after submission.

## Prerequisites

- Docker
- Kubernetes cluster (minikube, kind, or a cloud provider)
- kubectl configured to connect to your cluster
- Optional: IBM Cloud CLI and Container Registry access

## Building the Application

### Complete the Dockerfile

The application uses a multi-stage build process:

```dockerfile
FROM golang:1.18 AS builder

WORKDIR /app

COPY main.go .

RUN go mod init guestbook
RUN go mod tidy

RUN go build -o main main.go

FROM ubuntu:18.04

COPY --from=builder /app/main /app/guestbook

COPY public/index.html /app/public/index.html
COPY public/script.js /app/public/script.js
COPY public/style.css /app/public/style.css
COPY public/jquery.min.js /app/public/jquery.min.js

WORKDIR /app

CMD ["./guestbook"]
EXPOSE 3000
```

### Build and Push the Docker Image

```bash
# Set your registry namespace
export MY_NAMESPACE=your-registry-namespace

# Build the image
docker build -t us.icr.io/$MY_NAMESPACE/guestbook:v1 .

# Push to container registry
docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
```

## Deployment

### Deploy to Kubernetes

1. Modify the `deployment.yml` file with your registry namespace
2. Apply the deployment:

```bash
kubectl apply -f deployment.yml
```

3. Access the application:

```bash
kubectl port-forward deployment.apps/guestbook 3000:3000
```

Then visit `http://localhost:3000` in your browser.

## Autoscaling

The application can be configured to automatically scale based on CPU utilization:

```bash
kubectl autoscale deployment guestbook --cpu-percent=5 --min=1 --max=10
```

Monitor the autoscaling:

```bash
kubectl get hpa guestbook
```

## Rolling Updates

To perform a rolling update:

1. Update the application (e.g., modify `index.html`)
2. Build and push a new image version
3. Update the deployment YAML to use the new image
4. Apply the updated deployment

```bash
kubectl apply -f deployment.yml
```

## Rollbacks

View deployment history:

```bash
kubectl rollout history deployment/guestbook
```

Rollback to a previous revision:

```bash
kubectl rollout undo deployment/guestbook --to-revision=1
```

## License

[MIT](LICENSE)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
