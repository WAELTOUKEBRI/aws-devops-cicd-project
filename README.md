Project Documentation: Secure Automated Deployment and Observability Stack

Executive Summary

This project implements a comprehensive DevSecOps lifecycle on AWS, utilizing Infrastructure as Code (IaC), automated Continuous Integration/Continuous Deployment (CI/CD) pipelines, and a robust monitoring ecosystem. The primary objective was to move beyond simple container deployment toward a resilient, secure, and observable production environment.


Core Architectural Components
1. Infrastructure as Code (IaC)
The foundational cloud environment was provisioned using Terraform. This ensures that the networking layer—including the VPC, Subnets, and Security Groups—along with the EC2 compute resources, are version-controlled and reproducible.

2. Automated CI/CD Pipeline
GitHub Actions serves as the orchestration engine. The pipeline is designed to execute a series of high-stakes tasks upon every code commit:

Environment Setup: Preparation of the runner and acquisition of required dependencies.

Security Audit: Integration of Trivy to perform deep scans of Docker images for CVEs (Common Vulnerabilities and Exposures) prior to deployment.

Automated Deployment: Secure remote execution via SSH to update the EC2 instance and manage container lifecycles.

3. Application Resiliency
To ensure high availability, the deployment script incorporates Docker health checks. These checks monitor the internal state of the application, allowing the infrastructure to identify and automatically restart unresponsive containers through a defined restart: unless-stopped policy.

4. Observability and Infrastructure Governance
A dedicated monitoring stack provides real-time visibility into system health and performance:

Metrics Collection: Prometheus acts as the time-series database, scraping data from Node Exporter.

Data Visualization: Grafana provides high-fidelity dashboards for monitoring CPU utilization, memory consumption, and network throughput.

Cloud Alerting: Proactive infrastructure monitoring is handled by AWS CloudWatch Alarms, configured via the AWS CLI to trigger notifications when performance thresholds are breached.

Technical Specifications
Domain	Technology
Cloud Provider	Amazon Web Services (AWS)
Infrastructure	Terraform, IAM, CloudWatch
Containerization	Docker, Docker Network
CI/CD	GitHub Actions
Security Scanning	Trivy
Monitoring	Prometheus, Grafana, Node Exporter
Verification and Evidence
Pipeline Integrity
The GitHub Actions workflow demonstrates a successful execution of the security-gate and deployment logic.
(Ref: cicd2.png)

Security Scan Results
Trivy reports provide full visibility into the software supply chain, identifying vulnerabilities within the base image for risk management and patching.
(Ref: cicd5.png)

Real-Time Monitoring
The Grafana dashboard visualizes live metrics, confirming the successful integration of the Prometheus data source and the Node Exporter agent.
(Ref: cicd8.png)

Infrastructure Alerting
Terminal-based configuration of CloudWatch Alarms confirms the ability to manage AWS resources programmatically via the Command Line Interface (CLI).
(Ref: cicd14.png, cicd12.png)

Troubleshooting and Technical Challenges
Problem: IAM Authorization Failure
During the automation of CloudWatch Alarms via the AWS CLI, the process initially failed with a Credentials not found error despite the instance being within the AWS environment.

Solution: Identity and Access Management (IAM) Roles
Instead of using static credentials, an IAM Role was created with CloudWatchFullAccess permissions and attached to the EC2 instance. This leveraged the AWS Metadata Service to provide secure, temporary credentials, adhering to the principle of least privilege.

Deployment Instructions
Prerequisites
AWS Account with configured IAM permissions for Terraform and EC2.

Terraform installed on the local management machine.

GitHub repository secrets configured: EC2_SSH_KEY, EC2_PUBLIC_IP, and AWS credentials.

Execution
Initialize Infrastructure: Execute terraform apply within the infrastructure directory to provision the AWS resources.

Pipeline Activation: Push changes to the main branch to trigger the GitHub Actions workflow.

Access Monitoring: Access the Grafana interface by navigating to the EC2 public IP on port 3000.

Would you like me to help you create a "Future Roadmap" section for this README, detailing how you would add Auto-Scaling or a Load Balancer next?
