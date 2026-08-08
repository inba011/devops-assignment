# DevOps Engineer Take-Home Assignment

## Overview

This project is a two-service application that was reviewed, debugged, and improved to ensure reliable containerized deployment and communication between services.

The application consists of:

* **Service A** – Python/Flask API
* **Service B** – Node.js worker

Service B periodically polls Service A's `/data` endpoint.

The primary objective of this project was to identify and resolve issues across the application, Docker configuration, CI/CD configuration, Terraform configuration, and Kubernetes manifests, and to document the fixes and improvements.

---

## Architecture

```text
                    ┌──────────────────────┐
                    │      Service B       │
                    │     Node.js Worker   │
                    │                      │
                    │  Polls Service A     │
                    │  every 10 seconds    │
                    └──────────┬───────────┘
                               │
                               │ HTTP
                               ▼
                    ┌──────────────────────┐
                    │      Service A       │
                    │     Python / Flask   │
                    │                      │
                    │  /                   │
                    │  /health             │
                    │  /data               │
                    └──────────────────────┘
```

---

## Technologies Used

* Docker
* Docker Compose
* Python
* Flask
* Node.js
* Git
* GitHub
* GitHub Actions
* Terraform
* Kubernetes
* Linux
* REST API

---

## Project Structure

```text
.
├── service-a/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── service-b/
│   ├── worker.js
│   ├── package.json
│   └── Dockerfile
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── terraform/
│   └── main.tf
│
├── k8s/
│   └── deployment.yaml
│
├── docker-compose.yml
├── FIXES.md
└── README.md
```

---

## Services

### Service A – Flask API

Service A is a Python Flask application that exposes the following endpoints:

| Endpoint  | Description                         |
| --------- | ----------------------------------- |
| `/`       | Application endpoint                |
| `/health` | Health check endpoint               |
| `/data`   | Data endpoint consumed by Service B |

The service runs on port `5000`.

---

### Service B – Node.js Worker

Service B is a Node.js worker application.

It periodically communicates with Service A and polls the `/data` endpoint every 10 seconds.

The service uses the Docker Compose service network to communicate with Service A.

---

## Running the Application

### Prerequisites

Install the following before running the project:

* Docker
* Docker Compose
* Git

### Clone the Repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd <YOUR_REPOSITORY_NAME>
```

### Build and Start the Services

```bash
docker-compose up --build
```

Or, with newer Docker Compose versions:

```bash
docker compose up --build
```

### Run in Detached Mode

```bash
docker-compose up -d --build
```

### Check Running Services

```bash
docker-compose ps
```

### View Logs

```bash
docker-compose logs -f
```

### View Service A Logs

```bash
docker-compose logs -f service-a
```

### View Service B Logs

```bash
docker-compose logs -f service-b
```

---

## Testing Service A

After starting the containers, the Flask API can be tested using:

```bash
curl http://localhost:5000/
```

Health check:

```bash
curl http://localhost:5000/health
```

Data endpoint:

```bash
curl http://localhost:5000/data
```

---

## Service-to-Service Communication

Service B communicates with Service A through the Docker Compose network.

Instead of using:

```text
localhost
```

Service B communicates with Service A using its Compose service name.

This allows the containers to communicate correctly within the Docker network.

---

## Troubleshooting

The project involved identifying and resolving issues across multiple components, including:

* Application configuration
* Python dependencies
* Node.js dependencies
* Dockerfiles
* Docker Compose configuration
* Container networking
* Service-to-service communication
* Service startup dependencies
* Health checks
* CI/CD configuration
* Terraform configuration
* Kubernetes configuration

Detailed information about each identified issue is documented in:

```text
FIXES.md
```

Each documented issue includes:

* What was wrong
* Why it was a problem
* How it was fixed
* Potential impact if left unresolved

---

## Improvements

In addition to fixing the identified issues, additional improvements were considered and implemented where appropriate to improve:

* Service reliability
* Container startup behavior
* Health monitoring
* Service communication
* Configuration management
* Deployment consistency
* Documentation
* Maintainability

---

## CI/CD

The repository includes a GitHub Actions workflow:

```text
.github/workflows/deploy.yml
```

The workflow was reviewed and improved to support a more reliable CI/CD process.

---

## Terraform

Infrastructure configuration is maintained under:

```text
terraform/
```

The Terraform configuration was reviewed as part of the overall infrastructure and deployment troubleshooting process.

---

## Kubernetes

Kubernetes configuration is maintained under:

```text
k8s/
```

The Kubernetes manifest was reviewed for deployment configuration and consistency.

---

## Git Workflow

The project follows a logical Git commit history.

Each significant fix or improvement is maintained as a separate logical commit.

Example commit structure:

```text
fix: resolve service-a dependency issue
fix: correct service-b communication
fix: improve docker compose configuration
fix: correct ci workflow
improvement: add service health checks
docs: document troubleshooting fixes
```

---

## Validation

The final application can be started using:

```bash
docker-compose up --build
```

The expected result is:

1. Service A starts successfully.
2. Service A responds to its health endpoint.
3. Service B starts successfully.
4. Service B connects to Service A.
5. Service B periodically polls the `/data` endpoint.
6. Both services remain operational without recurring startup or communication failures.

---

## Documentation

Detailed troubleshooting and improvement documentation is available in:

```text
FIXES.md
```

The document provides the root cause, resolution, and potential impact of each issue identified during the project.

---

## Key DevOps Concepts Demonstrated

This project demonstrates practical knowledge of:

* Containerization
* Docker Compose
* Docker networking
* Multi-service application deployment
* Application troubleshooting
* Incident investigation
* Root-cause analysis
* Health checks
* CI/CD
* GitHub Actions
* Infrastructure as Code
* Terraform
* Kubernetes
* Linux
* Git version control
* Technical documentation
* Service reliability

---

## Author

**Inba Malar Vendhan A**

Technical Support Engineer | Systems Engineer | Cloud & DevOps
