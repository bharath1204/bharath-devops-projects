
# DevOps Monitoring and CI/CD Automation on AWS EC2

## 1. Project Overview

This project demonstrates an end-to-end DevOps workflow using AWS EC2, Jenkins, Docker, Flask, Prometheus, Grafana and GitHub.

The objective was to automate application deployment whenever code changes are pushed to GitHub and monitor application and server health using Prometheus and Grafana.

The project was implemented on an Ubuntu EC2 instance.

## 2. Architecture

GitHub Repository
       |
       | Git Push
       v
GitHub Webhook
       |
       v
Jenkins CI/CD Pipeline
       |
       |-- Checkout Source Code
       |-- Build Docker Image
       |-- Deploy Docker Container
       |-- Verify Application Health
       |
       v
Flask Application (Port 5000)
       |
       |-- /health
       |-- /metrics
       |
       v
Prometheus (Port 9090)
       |
       |-- Flask Application Metrics
       |-- Node Exporter Metrics
       |
       v
Grafana (Port 3000)

Node Exporter (Port 9100) collects EC2 system metrics.

## 3. Technologies Used

- AWS EC2
- Ubuntu Linux
- Git and GitHub
- Jenkins
- GitHub Webhooks
- Docker
- Python
- Flask
- Prometheus
- Grafana
- Node Exporter
- Bash

## 4. Project Structure

```text
devops-monitoring-project/
├── app.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
└── README.md
```

## 5. Flask Application

Created a Python Flask application with three endpoints.

### Home endpoint

`/`

Returns a test message and increments a custom Prometheus request counter.

### Health endpoint

`/health`

Returns:

```json
{"status": "healthy"}
```

Used by Jenkins to verify successful application deployment.

### Metrics endpoint

`/metrics`

Exposes application metrics in Prometheus format.

Created a custom counter:

`app_requests_total`

This tracks the total number of requests received by the Flask application.

## 6. Docker Containerisation

Created a Dockerfile using the Python 3.12 slim base image.

Main steps:

1. Select Python base image.
2. Set working directory.
3. Copy requirements.txt.
4. Install Python dependencies.
5. Copy application code.
6. Expose port 5000.
7. Start the Flask application.

Built Docker images with versioned tags using Jenkins build numbers.

Example:

`devops-monitoring-app:5`

## 7. Jenkins CI/CD Pipeline

Created a Jenkins Declarative Pipeline using Jenkinsfile.

### Stage 1: Build Docker Image

```bash
docker build -t devops-monitoring-app:$BUILD_NUMBER ./devops-monitoring-project
```

Builds a Docker image using the current Jenkins build number.

### Stage 2: Deploy Application

```bash
docker rm -f monitoring-app || true
```

Removes the previous application container if it exists.

```bash
docker run -d \
  --name monitoring-app \
  --restart unless-stopped \
  -p 5000:5000 \
  devops-monitoring-app:$BUILD_NUMBER
```

Starts the new application container.

### Stage 3: Verify Deployment

```bash
sleep 5
curl --fail http://localhost:5000/health
```

Checks whether the application is healthy after deployment.

Expected response:

```json
{"status": "healthy"}
```

## 8. GitHub Webhook Integration

Configured a GitHub webhook to trigger Jenkins automatically whenever code is pushed to the main branch.

### Configuration

1. Opened the GitHub repository.
2. Navigated to Settings → Webhooks.
3. Added the Jenkins webhook endpoint:

`http://EC2_PUBLIC_IP:8080/github-webhook/`

4. Selected `application/json`.
5. Selected Just the push event.
6. Enabled the webhook.

In Jenkins:

1. Opened the Devops-monitoring-project job.
2. Navigated to Configure → Build Triggers.
3. Enabled GitHub hook trigger for GITScm polling.

### Verification

GitHub successfully delivered a webhook with HTTP 200.

Pushed a test change to the main branch.

Jenkins automatically detected the GitHub push and started a new build without manually selecting Build Now.

## 9. Monitoring Setup

### Prometheus

Configured Prometheus to scrape:

- Flask application metrics.
- Node Exporter metrics.
- Prometheus server metrics.

Verified that all configured targets were UP.

### Grafana

Connected Prometheus as a Grafana data source.

Imported the Node Exporter Full dashboard.

Created a custom Flask Application Monitoring dashboard with:

1. Total Flask Requests.
2. Flask Request Rate.
3. Flask Application Status.
4. EC2 CPU Usage.
5. EC2 Memory Usage.
6. EC2 Disk Usage.

Used PromQL queries to visualise application and infrastructure metrics.

## 10. Errors, Mistakes and Troubleshooting

### Issue 1: Jenkins was not accessible through the browser

**Problem:**

The Jenkins website showed that the site could not be reached.

**Reason:**

The EC2 instance had been stopped and its public IP could change after restarting.

**Resolution:**

- Started the EC2 instance.
- Checked its current public IPv4 address.
- Established an SSH tunnel.
- Accessed Jenkins through localhost:8080.

### Issue 2: GitHub could not reach Jenkins through localhost

**Problem:**

Jenkins was initially accessible through an SSH tunnel, but GitHub could not send webhook requests to localhost.

**Reason:**

The SSH tunnel only provided access from my local computer. GitHub could not reach that local address.

**Resolution:**

- Checked Jenkins service status.
- Verified that Jenkins was listening on port 8080.
- Added a temporary EC2 security group inbound rule for TCP port 8080.
- Configured the GitHub webhook using the EC2 public IP.

**Learning:**

A GitHub webhook requires a reachable endpoint. Local SSH tunnels are not directly accessible to external webhook providers.

**Security consideration:**

The temporary port 8080 rule exposed the Jenkins web interface publicly over HTTP. It should be removed after testing and replaced with a secure HTTPS access method for continued webhook operation.

### Issue 3: Jenkins Git installation warning

**Warning:**

`Selected Git installation does not exist. Using Default`

**Reason:**

The configured Git tool installation was not found by Jenkins.

**Result:**

Jenkins successfully used the default Git installation and checked out the repository.

**Learning:**

A Jenkins warning does not always mean the pipeline has failed. Check the actual command results and build status.

### Issue 4: Docker legacy builder deprecation warning

**Warning:**

`The legacy builder is deprecated`

**Reason:**

Docker's legacy image builder is being phased out.

**Result:**

The Docker image was still built successfully.

**Future improvement:**

Configure Docker Buildx and BuildKit.

## 11. Final Pipeline Verification

GitHub commit:

`Test GitHub webhook trigger`

Jenkins build:

`Build #5`

Final status:

`Finished: SUCCESS`

Verified results:

- GitHub push triggered Jenkins automatically.
- Jenkins checked out the latest source code.
- Docker image was built successfully.
- Previous application container was replaced.
- New application container started successfully.
- Health check returned `{"status": "healthy"}`.

## 12. Key Learnings

Through this project, I gained practical experience in:

- Linux server administration.
- AWS EC2 instance management.
- Git and GitHub version control.
- Jenkins Declarative Pipelines.
- GitHub webhook integration.
- Docker image building and container deployment.
- Flask application development.
- Prometheus metrics collection.
- Grafana dashboard creation.
- Infrastructure and application monitoring.
- CI/CD troubleshooting.
- Debugging deployment and connectivity issues.

## 13. Future Improvements

- Secure Jenkins using HTTPS and a reverse proxy.
- Implement a persistent and secure GitHub webhook endpoint.
- Configure Docker BuildKit.
- Add automated application tests to the pipeline.
- Introduce Terraform for AWS infrastructure provisioning.
- Introduce Ansible for server configuration.
- Explore Kubernetes and Helm deployment.

---

**Project Status:** Completed — GitHub-triggered Jenkins CI/CD pipeline and application monitoring successfully demonstrated.
