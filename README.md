# Study Timer

A quiet, customizable timer for focused study sessions and proper breaks. It is a standalone static web app: no build step, account, server-side database, or external service is required.

## Features

- Adjustable focus and break intervals with automatic switching
- Four themes: Focus and Calm, each in light and dark
- Optional interval chime and auto-start
- Daily session count and focus time
- Keyboard shortcuts: **Space** to start or pause, **R** to reset, and **N** to skip
- Preferences and progress saved in the current browser

## Run locally

Serve the repository root with any static file server. For example, with Python:

```sh
python3 -m http.server 8000
```

Then open <http://localhost:8000>. The app is served directly from `index.html`; there is no install or build command.

## Deploy as a static site

Copy `index.html` and the `assets/` directory to the document root of any static web host. Set the publish directory to the repository root and leave the build command empty.

## Deploy with Docker

Build the image:

```sh
docker build -t study-timer:local .
```

Run it on port 8080:

```sh
docker run --rm --name study-timer \
  -p 8080:8080 \
  --read-only \
  --tmpfs /tmp \
  --cap-drop=ALL \
  --security-opt=no-new-privileges:true \
  study-timer:local
```

Open <http://localhost:8080>. The container uses the unprivileged NGINX image, which runs as a non-root user and listens on port 8080 by default. See the [upstream image documentation](https://github.com/nginx/docker-nginx-unprivileged) for image details and production pinning guidance.

### Docker Compose

```sh
docker compose up --build -d
```

The app is available at <http://localhost:8080>. Stop it with:

```sh
docker compose down
```

## Deploy to Kubernetes

The manifest in `k8s/study-timer.yaml` creates a namespace, a two-replica Deployment, and a ClusterIP Service. Build and push the image to a registry your cluster can access:

```sh
docker build -t ghcr.io/YOUR_USER/study-timer:1.0.0 .
docker push ghcr.io/YOUR_USER/study-timer:1.0.0
```

Edit the `image` value in `k8s/study-timer.yaml` to match that image, then apply the manifest:

```sh
kubectl apply -f k8s/study-timer.yaml
kubectl rollout status deployment/study-timer -n study-timer
kubectl port-forward -n study-timer service/study-timer 8080:80
```

Open <http://localhost:8080>. For public access, route your cluster's Ingress or Gateway to the `study-timer` Service on port 80 and configure TLS with your cluster's certificate setup. If the image registry is private, configure an image pull secret in the namespace.

## Data and privacy

The app stores timer preferences and daily progress in the browser's `localStorage`. Data stays in that browser profile and does not sync between devices. The app has no backend, user accounts, analytics, or third-party runtime dependencies.

## Repository layout

```text
.
├── assets/
│   └── icon.svg
├── k8s/
│   └── study-timer.yaml
├── Dockerfile
├── compose.yaml
├── index.html
└── nginx.conf
```
