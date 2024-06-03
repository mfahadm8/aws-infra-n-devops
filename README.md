# Project Infrastructure and CI/CD Pipeline Setup

This document outlines the design details for the infrastructure configuration and deployment of the project, focusing on infrastructure optimization, automation, and CI/CD pipeline setup. The infrastructure will be managed using AWS Cloud Development Kit (CDK), ensuring a streamlined, automated deployment process.

## 1. Infrastructure Optimization and Automation

### Infrastructure Provisioning

**Django Web App and PG Admin:**
- **ECS/Fargate:** We will use ECS/Fargate instead of EC2 instances to host our website, as it is lighter, more scalable, and more secure.
- **Communication:** Communication between Django and other services will use private hosted zones.
- **VPC:** Both Django and other components will reside within the same VPC to facilitate communication.

### Infrastructure Configuration

**Choice of CDK over CloudFormation/Terraform:**
- **CDK:** CDK will be used for defining the infrastructure due to its flexibility in using a programming language and providing numerous Level 3 (L3) constructs that reduce the amount of code required.

**Parameter Management:**
- **SSM Parameters:** SSM parameters will be used to avoid hardcoding values or passing infrastructure IDs directly. For example:
  - `/megadolls/infra/vpc_id` for VPC ID.
  - `/megadolls/infra/db_url` for database URL.
  - `/megadolls/infra/strip_key` for business keys and secrets.

**Container Orchestration:**
- **Fargate ECS:** Fargate ECS will be utilized for container orchestration, allowing us to run containers without managing the underlying infrastructure.

### Environment Configuration

**Development (dev) Environment:**
- **Region:** us-east-1
- **ECS Cluster Configuration:** Fargate Spot (up to 70% discount, but can be restarted by AWS anytime)
  - **CPU:** 1 vCPU
  - **Memory:** 2 GB
  - **Autoscaling:** Enabled

**Production (prod) Environment:**
- **Region:** af-south-1
- **ECS Cluster Configuration:** Fargate (with at least one container using Simple Fargate Configuration to avoid interruptions)
  - **CPU:** 1 vCPU
  - **Memory:** 2 GB
  - **Autoscaling:** Enabled

### Infrastructure Layering

**Core Layer:**
- Contains VPC and other configurations less prone to frequent changes, providing a solid foundation for the infrastructure.

**Microservices Layer:**
- Consists of ECS definitions and microservices, responsible for running and managing the application's services.

**Deployment Layer:**
- Contains the infrastructure code for the pipeline to automatically deploy dev and prod services on new Docker image push.

## 2. CI/CD Pipeline Setup

### Introduction

This document outlines the design details for the deployment of the project using AWS Pipeline to automate the entire process.

### Deployment Pipeline

**Pipeline Stages:**

**Source:**
- ECR Repository with `latest` and `dev-latest` tags for prod and dev environments respectively.

**Manual Approval:**
- For prod deployments, an additional stage will require Super Admin approval to release changes to production.

**Build Phase:**
- Prepare required files like task definitions and image definition files for CodeDeploy Stage.

### Canary Deployment Configuration (Future)

**Production Environment:**
- Incremental release in steps of 10%, with observation periods to monitor for potential issues.

**Dev Environment:**
- Entire dev environment updated at once.

**Lambda Based Light-Weight Feature Release Flow:**
- A Lambda function will trigger deployments upon updates to the `latest` and `dev-latest` tags in the ECR repository.

## 3. Developer Workflow

### Dev/Staging Flow

- Developers work on feature branches and create pull requests to automatically deploy new features to the dev cluster for testing.
- Updates to the feature branch trigger redeployment to the dev cluster.

### Master/Prod Flow

- To release features to prod, developers create a pull request against the main branch, which is peer-reviewed and manually merged.
- Once the pipeline is fully implemented, manual approval gates will be added for production deployments.

### Image Tagging and Push to ECR

**Tagging Strategy:**
- **Dev Branch:** Tagged as `dev-latest`.
- **Prod Branch:** Tagged as `latest`.
- **Commit Tagging:** Each Git commit generates a tagged image with the commit ID.

### Image Cleanup Automation

- AWS Lambda and EventBridge rules will be used to delete images older than two months, excluding the `dev-latest` and `latest` images.

### Infrastructure as Code for CodeBuild Project

- No manual creation of resources or projects; infrastructure as code will be used for the entire project, allowing for easy recovery and reconfiguration.

## Conclusion

This setup ensures a robust, automated, and scalable infrastructure for the project, with a clear CI/CD pipeline to streamline deployments and manage changes efficiently. For any issues or setup, refer to the commands and instructions in the Repository Readme file to revert everything to normal.
