# test2 — Nginx Container CI/CD Lab

A hands-on DevOps lab demonstrating containerized web application deployment using Docker, GitHub Actions CI/CD pipelines, and remote server provisioning — tested across Ubuntu and RHEL 10 environments.

---

## What This Repo Does

Packages a static HTML page into an Nginx Docker container and automates its build, test, and deployment through four GitHub Actions workflows covering different real-world deployment scenarios.

---

## Repository Structure
test2/
├── Dockerfile                        # Nginx container definition
├── index.html                        # Static page served by Nginx
└── .github/workflows/
├── deploy-nginx.yml              # SSH-based remote server deployment
├── deploy-nginx-container.yml    # Docker Hub build, push, and verify
├── rhel10-lab-test.yml           # Self-hosted RHEL 10 runner validation
└── test-ssh-connection.yml       # SSH connectivity and key validation

---

## Dockerfile

```dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/
```

Minimal, production-pattern image — base nginx with custom HTML injected at build time. No unnecessary layers.

---

## GitHub Actions Workflows

### 1. deploy-nginx.yml — SSH Remote Deployment
Triggers on every push. SSHes into a remote server using stored secrets (SERVER_IP, SERVER_USER, SSH_PRIVATE_KEY), pulls latest nginx image, stops and removes any existing container, and runs a fresh container on port 80. Simulates a real bare-metal or VM deployment pattern.

**Secrets required:**
- `SERVER_IP` — target server IP
- `SERVER_USER` — SSH username
- `SSH_PRIVATE_KEY` — private key for authentication

---

### 2. deploy-nginx-container.yml — Docker Hub Pipeline
Triggers on push to main branch. Full CI/CD sequence: checkout → Docker Hub login → image build tagged as vikasdh/nginx:latest → container run on port 8080 → container health verification via docker ps and docker logs → live HTTP test via curl http://localhost:8080. Proves the built image actually serves content before any deployment.

**Secrets required:**
- `DOCKER_USERNAME`
- `DOCKER_PASSWORD`

---

### 3. rhel10-lab-test.yml — RHEL 10 Self-Hosted Runner
Triggers on every push. Runs on a self-hosted GitHub Actions runner registered on a RHEL 10 machine. Executes cat /etc/redhat-release to verify OS version and logs output — validates that the self-hosted runner is connected and RHEL 10 environment is functional. Foundation for extending lab tests to RHEL-specific tooling (dnf, systemd, SELinux).

**Requires:** Self-hosted runner registered with label self-hosted on a RHEL 10 machine.

---

### 4. test-ssh-connection.yml — SSH Key Validation
Triggers on push to main. Sets up SSH key from secrets, adds github.com to known_hosts via ssh-keyscan, and tests live SSH connectivity to a target server — printing confirmation message on success. Used to validate SSH secret configuration before running deployment workflows.

**Secrets required:**
- `SSH_PRIVATE_KEY`

---

## How to Run Locally

```bash
# Build the image
docker build -t vikasdh/nginx:latest .

# Run the container
docker run -d --name nginx-container -p 8080:80 vikasdh/nginx:latest

# Verify it is serving
curl http://localhost:8080
# Expected output: Hello, Nginx!

# Stop and clean up
docker stop nginx-container && docker rm nginx-container
```

---

## GitHub Secrets Required

| Secret | Used By | Purpose |
|---|---|---|
| SERVER_IP | deploy-nginx.yml | Remote server IP address |
| SERVER_USER | deploy-nginx.yml | SSH login username |
| SSH_PRIVATE_KEY | deploy-nginx.yml, test-ssh | Private key for SSH auth |
| DOCKER_USERNAME | deploy-nginx-container.yml | Docker Hub login |
| DOCKER_PASSWORD | deploy-nginx-container.yml | Docker Hub password |

---

## Lab Coverage

| Skill | Demonstrated |
|---|---|
| Docker image build | Dockerfile + docker build |
| Container lifecycle management | stop, rm, run pattern |
| GitHub Actions CI/CD | Four distinct workflow patterns |
| Remote SSH deployment | appleboy/ssh-action |
| Docker Hub registry | Login, tag, push |
| Container health verification | docker ps, docker logs, curl |
| Self-hosted runner RHEL 10 | rhel10-lab-test.yml |
| SSH key management | Secret injection, known_hosts |

---

## Author

Vikas Dhamija — Senior DevOps Engineer
GitHub: https://github.com/Vikas-DevOps-Git
