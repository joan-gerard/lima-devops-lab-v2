# lima-lab-v2

`lima-lab-v2` is a lightweight lab project that runs a static Nginx web page and a PostgreSQL database using Docker Compose, designed to be used inside a Lima Ubuntu VM on macOS.

## Project Contents

- `index.html`: Static page served by Nginx.
- `Dockerfile`: Builds the web image from `nginx:latest` and copies `index.html`.
- `compose.yaml`: Defines two services:
  - `web` (`build: .`) exposed on port `80`.
  - `db` (`postgres:latest`) with environment-based credentials and a persistent named volume.
- `lima-lab-v2.yaml`: Lima VM definition (Ubuntu 24.04 ARM64, Docker enabled, mounts home directory).
- `.github/workflows/deploy.yml`: GitHub Actions workflow that builds and pushes the Docker image to Docker Hub on pushes to `main`.

## Architecture

- **Web service**: Nginx serves a static HTML page.
- **Database service**: PostgreSQL runs as a separate container with persistent storage.
- **Runtime environment**: Intended to run in a Lima VM with Docker installed via provisioning.
- **CI/CD**: GitHub Actions builds and pushes image tag `latest` to Docker Hub.

## Prerequisites

- macOS with [Lima](https://lima-vm.io/) installed
- Docker CLI / Compose support available in the VM
- Git

## Running Locally with Lima

1. Start a Lima instance from the provided config:

   ```bash
   limactl start lima-lab-v2.yaml
   ```

2. Open a shell in the VM:

   ```bash
   limactl shell lima-lab-v2
   ```

3. Move to this repository inside the mounted directory and start services:

   ```bash
   docker compose up --build -d
   ```

4. Verify running containers:

   ```bash
   docker compose ps
   ```

5. Open the app in your browser:
   - `http://localhost` (if port forwarding and host networking are available in your setup)

## Service Configuration

From `compose.yaml`:

- **Web**
  - Port mapping: `80:80`
  - Restart policy: `always`
- **Database**
  - Image: `postgres:latest`
  - Environment:
    - `POSTGRES_USER=devuser`
    - `POSTGRES_PASSWORD=devpass`
    - `POSTGRES_DB=devdb`
  - Volume:
    - `postgres_data:/var/lib/postgresql`
  - Restart policy: `always`

## CI/CD (GitHub Actions)

On every push to `main`, `.github/workflows/deploy.yml`:

1. Checks out the repository.
2. Logs in to Docker Hub using repository secrets:
   - `DOCKER_USERNAME`
   - `DOCKER_PASSWORD`
3. Builds and pushes the Docker image with tag:
   - `${DOCKER_USERNAME}/joan-v2-nginx:latest`

## Notes

- This repository currently serves static content only.
- Database credentials in `compose.yaml` are suitable for local/lab usage and should be replaced for production scenarios.
