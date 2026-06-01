# Timestamp API — Candidate Exercise

## Overview

Develop starter Go API service. Your job is to complete it, containerize it, and deploy it to a local Kubernetes cluster using **kind**.

---

## What the API should do

- Expose a `GET /timestamp` endpoint
- On every request, generate a **random alphanumeric string** of a **random length**
- Log to stdout: the **current UTC timestamp** + the **random string** + its **length**
- Return a JSON response containing the timestamp, unix time, and the random string

**Example log line:**
```
2024/01/15 10:23:45 [2024-01-15T10:23:45Z] GET /timestamp | random_string=aB3xQr9 (len=7) | from=127.0.0.1:54321
```

**Example JSON response:**
```json
{
  "timestamp": "2024-01-15T10:23:45Z",
  "unix": 1705314225,
  "random_string": "aB3xQr9"
}
```

---

## TODO

### 1. Complete the Go API

- [ ] In `main.go`, implement the `randomString()` function
  - Generate a **random length** between 5 and 14
  - Fill it with random **alphanumeric characters** (a–z, A–Z, 0–9)
- [ ] In the `/timestamp` handler, call `randomString()` and include it in both the **log line** and the **JSON response**
- [ ] Make sure the `/health` endpoint returns `{"status":"ok"}`
- [ ] Run locally and verify: `go run main.go`

---

### 2. Containerize with Docker

- [ ] Write a `Dockerfile` using a **multi-stage build**
  - Stage 1: use `golang:1.21-alpine` to build the binary
  - Stage 2: use `alpine:3.18` to run it (keep the image small)
- [ ] Build the image and tag it as `timestamp-api:latest`
  ```bash
  docker build -t timestamp-api:latest .
  ```
- [ ] Smoke test the container locally
  ```bash
  docker run -p 8080:8080 timestamp-api:latest
  curl http://localhost:8080/timestamp
  ```

---

### 3. Deploy to a kind cluster

- [ ] Create a local kind cluster
  ```bash
  kind create cluster --name demo
  ```
- [ ] Load your Docker image into the kind cluster (kind cannot pull local images without this)
  ```bash
  kind load docker-image timestamp-api:latest --name demo
  ```
- [ ] Write a `k8s.yaml` with a `Deployment` and a `Service`
  - Deployment: 1 replica, image `timestamp-api:latest`, `imagePullPolicy: Never`
  - Service: type `NodePort`, port `8080`, nodePort `30080`
- [ ] Apply the manifest
  ```bash
  kubectl apply -f k8s.yaml
  ```
- [ ] Verify the pod is running
  ```bash
  kubectl get pods
  kubectl logs -f <pod-name>
  ```

---

### 4. Test end to end

- [ ] Hit the API through the kind cluster
  ```bash
  kubectl port-forward svc/timestamp-api 8080:8080
  curl http://localhost:8080/timestamp
  curl http://localhost:8080/health
  ```
- [ ] Confirm the log in the pod output shows the timestamp and random string on every request

---

## Project structure (expected)

```
timestamp-api/
├── main.go         # Go source — complete this
├── go.mod          # Already provided
├── Dockerfile      # Write this
└── k8s.yaml        # Write this
```

---

## Tools you will need

| Tool | Install |
|------|---------|
| Go 1.21+ | https://go.dev/dl |
| Docker | https://docs.docker.com/get-docker |
| kind | `go install sigs.k8s.io/kind@latest` |
| kubectl | https://kubernetes.io/docs/tasks/tools |
