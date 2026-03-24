<div align="center">
  <h1 align="center">🚀 AWS DevSecOps CI/CD Pipeline</h1>
  <h3>Comprehensive Cloud-Native Deployment & Observability Stack</h3>

  <p align="center">
    <img src="https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white" alt="AWS" />
    <img src="https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white" alt="Terraform" />
    <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" />
    <img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white" alt="GitHub Actions" />
    <img src="https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white" alt="Prometheus" />
    <img src="https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white" alt="Grafana" />
    <img src="https://img.shields.io/badge/Trivy-1B2F4E?style=for-the-badge&logo=security&logoColor=white" alt="Trivy" />
  </p>
</div>

---

## 📖 Executive Summary

This project implements a fully automated, end-to-end **DevSecOps lifecycle** on AWS. Moving beyond simple container deployments, this architecture showcases a resilient, secure, and observable production environment. 

It leverages **Infrastructure as Code (IaC)** for reproducible cloud environments, **Continuous Integration/Continuous Deployment (CI/CD)** for zero-downtime updates, and a robust **Monitoring ecosystem** to ensure high availability and performance visibility.

---

## 🏗️ Core Architectural Components

### 1. Infrastructure as Code (IaC)
The foundational cloud environment is codified and provisioned using **Terraform**. This ensures the entire networking layer and compute resources are version-controlled, reproducible, and scalable.
* **VPC & Subnets:** Custom networking architecture.
* **Security Groups:** Granular traffic control (Ports 22, 80 open).
* **EC2 Compute:** Automatically provisioned instances with Amazon Linux 2.

### 2. Automated CI/CD Pipeline
**GitHub Actions** serves as the orchestration engine, executing automated workflows upon every code commit to the `main` branch.
* **Environment Setup:** Runner preparation and dependency acquisition.
* **Security Audit (DevSecOps):** Deep scanning of Docker images for CVEs (Common Vulnerabilities and Exposures) using **Trivy** prior to deployment.
* **Automated Deployment:** Secure remote execution via SSH to update the EC2 instance, manage container lifecycles, and handle zero-downtime container replacement.

### 3. Application Resiliency
To guarantee high availability, the deployment strategy incorporates advanced container orchestration on a single node:
* **Docker Health Checks:** Granular monitoring of application state (`curl -f http://localhost/ || exit 1`).
* **Auto-Recovery:** Infrastructure automatically identifies and restarts unresponsive containers via `restart: unless-stopped` policies.
* **Dedicated Networks:** A custom Docker network (`monitoring-network`) isolates component communication safely.

### 4. Observability and Infrastructure Governance
A dedicated, industry-standard monitoring stack provides real-time visibility into system health, performance, and anomalies.
* **Metrics Collection:** **Prometheus** aggregates timeseries data scraped via **Node Exporter**.
* **Data Visualization:** **Grafana** provides high-fidelity, interactive dashboards for CPU utilization, memory consumption, and network throughput.
* **Cloud Alerting:** Proactive monitoring executed by **AWS CloudWatch Alarms** (e.g., `EC2_High_CPU_Alert_DevOps_Project`), dynamically reacting when critical thresholds (like CPU Utilization > 80%) are breached.

---

## ⚙️ Technical Specifications

| Domain | Technology / Tooling |
| :--- | :--- |
| **Cloud Provider** | Amazon Web Services (AWS) |
| **Infrastructure (IaC)** | Terraform, IAM, CloudWatch |
| **Containerization** | Docker, Docker Compose / Network |
| **CI/CD** | GitHub Actions |
| **Security Scanning**| Trivy |
| **Monitoring Stack** | Prometheus, Grafana, Node Exporter |

---

## 🚀 Deployment Instructions

### Prerequisites
1. **AWS Account** with proper IAM permissions for EC2 and VPC management.
2. **Terraform CLI** installed on your local control machine.
3. Configure the following **GitHub Repository Secrets**:
   * `DOCKER_USERNAME` & `DOCKER_PASSWORD` *(Docker Hub registry authentication to prevent pull-rate limiting)*
   * `EC2_PUBLIC_IP`: The public IPv4 address yielded by Terraform outputs.
   * `EC2_SSH_KEY`: The private SSH key strictly matching the provisioned `aws_key_pair` (e.g., `devops_demo_key`).
   * AWS Credentials (if programmatic CloudWatch setup is required).

### Execution

1. **Initialize & Provision Infrastructure (IaC):**
   Navigate into the `terraform` directory to formally provision the AWS environment.
   ```bash
   cd terraform
   terraform init
   terraform validate  # (Optional: confirm configuration is strictly valid)
   terraform apply
   ```
   > [!NOTE]
   > Upon a successful apply, Terraform explicitly prints the `server_public_ip` (e.g., `35.180.33.192`) in terminal output. Update your local configuration and GitHub Secrets with this IP address.

2. **Pipeline Activation:**
   Push code changes to the `main` branch. The GitHub Actions workflow will automatically trigger, build, scan, and deploy the application.

3. **Access Monitoring Systems:**
   * **Application:** `http://<EC2-PUBLIC-IP>:80`
   * **Grafana:** `http://<EC2-PUBLIC-IP>:3000` (Visualize Dashboards)
   * **Prometheus:** `http://<EC2-PUBLIC-IP>:9090` (Query Metrics)

4. **Verify Application Resiliency Locally (Optional):**
   Establish a secure shell session mapping an authorized ECDSA/RSA key to inspect running daemon processes on Amazon Linux 2:
   ```bash
   ssh -i ~/.ssh/devops_demo_key ec2-user@<server_public_ip>
   sudo docker ps
   ```

---

## 🛡️ Verification and Evidence

The pipeline integrates multiple checkpoints ensuring quality, security, and stability.

| Feature | Description |
| :--- | :--- |
| **Pipeline Integrity** | The GitHub Actions workflow demonstrates a successful execution of both security-gate and deployment logic to the remote hosts. |
| **Security Scanning** | Trivy reports provide complete software supply chain transparency, identifying both OS and library vulnerabilities for proactive patching. |
| **Real-Time Monitoring** | Grafana dashboards actively ingest Prometheus data points, displaying system performance intuitively. |
| **Infrastructure Alerting** | **CloudWatch Alarms** (e.g., `EC2_High_CPU_Alert_DevOps_Project`) are meticulously configured via the AWS CLI to manage infrastructure reactively, notifying operators on high resource strain (CPU > 80%). |

---

## 🔧 Troubleshooting and Technical Challenges

### Problem: Programmatic CloudWatch Alarm Configuration
During the automation of CloudWatch Alarms via the AWS CLI directly from the EC2 instance, the process inherently blocked programmatic access, throwing a `Credentials not found` error despite operating entirely within the AWS ecosystem.

### Solution: IAM Roles & AWS IMDSv2 (Security Posture)
Instead of passing vulnerable static credentials, an **IAM Role** with `CloudWatchFullAccess` permissions was bound to the EC2 instance profile. Furthermore, the architecture adheres to strict security principles by securely interacting with the **AWS Instance Metadata Service v2 (IMDSv2)**:

```bash
# 1. Generate IMDSv2 secure token
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# 2. Extract specific Instance ID
INSTANCE_ID=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)

# 3. Provision CloudWatch Alarm programmatically
aws cloudwatch put-metric-alarm \
  --alarm-name "CPU_High_Alert_Terminal" \
  --alarm-description "Alarm when CPU exceeds 80 percent" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=$INSTANCE_ID \
  --evaluation-periods 1 \
  --unit Percent
```
This strategy ensures zero-trust connectivity while dynamically provisioning alarms across any instantiated node.

---
<div align="center">
  <b>Built with ❤️ using Cloud-Native Technologies.</b>
</div>
