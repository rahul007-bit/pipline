# Flask Application with Kubernetes Deployment

A simple Flask application with Kubernetes deployment manifests and GitHub Actions CI/CD pipeline.

## Project Structure

```
.
├── app.py                    # Flask application
├── requirements.txt          # Production dependencies
├── requirements-dev.txt      # Development dependencies (includes pytest)
├── Dockerfile                # Docker image configuration
├── .dockerignore             # Docker ignore file
├── tests/
│   └── test_app.py          # Unit tests
├── k8s/
│   ├── deployment.yaml      # Kubernetes Deployment manifest
│   └── service.yaml         # Kubernetes Service manifest
└── .github/
    └── workflows/
        ├── pr.yaml          # GitHub Actions workflow for PRs
        └── release.yaml     # Build, push, and release workflow
```

## Local Development

### Prerequisites
- Python 3.11+
- Docker (optional)
- kubectl and a Kubernetes cluster (for deployment)

### Running Locally

1. Create a virtual environment:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements-dev.txt
   ```

3. Run the application:
   ```bash
   python app.py
   ```

4. Access the application at `http://localhost:5000`

### Running Tests

```bash
pytest tests/ -v
```

## Docker

### Build the image

```bash
docker build -t flask-app:latest .
```

### Run the container

```bash
docker run -p 5000:5000 flask-app:latest
```

## Kubernetes Deployment

### Prerequisites
- A running Kubernetes cluster
- kubectl configured to connect to your cluster
- Docker image pushed to a container registry

### Deploy to Kubernetes

1. Apply the manifests:
   ```bash
   kubectl apply -f k8s/deployment.yaml
   kubectl apply -f k8s/service.yaml
   ```

2. Check the deployment status:
   ```bash
   kubectl get pods -l app=flask-app
   kubectl get svc flask-app-service
   ```

## API Endpoints

| Endpoint  | Method | Description                    |
|-----------|--------|--------------------------------|
| `/`       | GET    | Returns a hello world message  |
| `/health` | GET    | Health check endpoint          |
| `/ready`  | GET    | Readiness check endpoint       |

## GitHub Actions

The project includes two GitHub Actions workflows:

### PR Validation (`.github/workflows/pr.yaml`)
Runs on pull requests to the `main` branch:
1. **Test Job**: Runs pytest to execute unit tests
2. **Lint Job**: Runs flake8 for code linting
3. **Build Job**: Builds the Docker image (after tests and lint pass)

### Build and Release (`.github/workflows/release.yaml`)
Runs on pushes to `main` and version tags:

1. **Build and Push**: Builds and pushes Docker image to GitHub Container Registry (ghcr.io)
2. **Update K8s Manifests**: Automatically updates `k8s/deployment.yaml` with the new image tag

## Docker Image Tagging Strategy

Images are pushed to `ghcr.io/rahul007-bit/pipline` with the following tags:

| Trigger | Tags Generated |
|---------|----------------|
| Push to `main` | `latest`, `sha-<commit>` |
| Tag `v1.2.3` | `1.2.3`, `1.2`, `1`, `sha-<commit>` |

### Creating a New Release

To release a new version:

```bash
# Create and push a semantic version tag
git tag v1.0.0
git push origin v1.0.0
```

This will:
1. Build and push the Docker image with version tags
2. Automatically update `k8s/deployment.yaml` with the new image
3. Commit the changes back to `main`

## License

MIT
