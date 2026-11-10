# AWS Two-Tier Architecture with Terraform
<img width="490" height="383" alt="image" src="https://github.com/user-attachments/assets/c21e3773-ed32-445e-bfd3-f0f88e328006" />


## 📌 Overview

This project demonstrates a **Two-Tier Web Application Architecture on AWS** using **Terraform** for Infrastructure as Code (IaC). It follows a modular approach to provision infrastructure with an emphasis on security, maintainability, and cost awareness.

The architecture includes a public-facing application tier and a private database tier. A single EC2 instance hosts the application, an Application Load Balancer (ALB) handles incoming traffic, and Amazon RDS provides managed database services.

## ✅ Key Features

- **Modular Architecture** – Reusable Terraform modules for organized infrastructure management
- **Infrastructure as Code (IaC)** – Provision AWS resources consistently using Terraform
- **Networking** – Custom VPC with two public and two private subnets across Availability Zones
- **Security** – IAM roles and security groups following least-privilege principles
- **Compute** – One EC2 instance deployed in a public subnet
- **Load Balancing** – Application Load Balancer routes web traffic to EC2
- **Database Integration** – One Amazon RDS instance deployed in private subnets
- **CDN & DNS ** – CloudFront, ACM, and Route 53 for content delivery, HTTPS, and DNS
- **Cost-conscious design** – No NAT Gateways, Elastic IPs, Auto Scaling Group, or WAF

---

## 🏗️ Architecture

![AWS Two-Tier Architecture](architecture.png)

### Architecture Components

| Component | Purpose |
|---|---|
| Amazon VPC (`10.0.0.0/16`) | Isolated network for project resources |
| Public Subnet 1 (`10.0.1.0/24`) | Hosts the single EC2 instance |
| Public Subnet 2 (`10.0.2.0/24`) | Second public subnet for ALB placement |
| Private Subnet 1 (`10.0.3.0/24`) | RDS subnet group subnet |
| Private Subnet 2 (`10.0.4.0/24`) | RDS subnet group subnet in another AZ |
| Internet Gateway | Internet connectivity for public subnet routes |
| EC2 | Runs the web/application tier |
| ALB | Receives and forwards HTTP/HTTPS requests to EC2 |
| RDS | Managed database; not publicly accessible |
| IAM | EC2 role and instance profile for AWS permissions/SSM |
| CloudFront  | CDN for content delivery |
| Route 53  | DNS records and domain routing |

### Request Flow

1. A user accesses the application through the ALB DNS name or configured domain.
2. (Optional) Route 53 resolves the domain, and CloudFront can deliver/cache content.
3. The ALB forwards requests to the EC2 instance registered in its target group.
4. The application on EC2 connects to RDS using its private endpoint.
5. RDS security group accepts database traffic only from the EC2 security group.

> **Note:** A single EC2 instance is not compute high availability. The ALB spans two public subnets, but the application currently has one backend instance.

---

## 🚀 Getting Started

### Prerequisites

- **Terraform** >= 1.10.0 (for S3 native state locking)
- **AWS CLI v2** configured with appropriate credentials
- An **AWS account** with permissions to create the required resources
- An existing S3 bucket for Terraform remote state
- A registered domain name if using Route 53 / ACM / CloudFront

### 1. Clone the Repository

```bash
git clone <REPOSITORY-URL>
cd <REPOSITORY-DIRECTORY>
```

### 2. Configure Variables

Before deployment, update `variables.tfvars`:

- Set a secure `RDS-PWD` (never commit credentials)
- Set `DOMAIN-NAME` to a domain you own if using DNS/CDN modules
- Review CIDR blocks, resource names, AMI selection, and region
- Confirm all values match the inputs expected by your modules

Keep your real tfvars file out of Git. Add it to `.gitignore` and optionally commit a sanitized `variables.tfvars.example`.

### 3. Configure AWS Credentials

```bash
aws configure
aws sts get-caller-identity
```

Confirm that the AWS identity and region are correct before proceeding.

### 4. Initialize Terraform

```bash
terraform init
```

### 5. Format and Validate

```bash
terraform fmt -recursive
terraform validate
```

### 6. Review and Apply

```bash
terraform plan -var-file="variables.tfvars"
terraform apply -var-file="variables.tfvars"
```

Review the plan and type `yes` when prompted. Avoid `--auto-approve` until you are confident about the resources and charges.

### 7. Cleanup (Destroy Infrastructure)

```bash
terraform plan -destroy -var-file="variables.tfvars"
terraform destroy -var-file="variables.tfvars"
```

> **Warning:** Destroying infrastructure can permanently delete resources and database data. Back up anything important first.

---

## 🔧 Configuration Files

- **`backend.tf`** – Terraform S3 backend and Terraform/provider version requirements
- **`main.tf`** – Root Terraform configuration and module calls
- **`variables.tf`** – Input variable declarations
- **`variables.tfvars`** – Values for the declared variables (local, sensitive)
- **`outputs.tf`** – Useful outputs such as ALB DNS name, EC2 ID, and RDS endpoint (avoid exposing secrets)
- **`modules/`** – Reusable Terraform modules

### Module Structure

```text
modules/
├── aws-vpc/
├── security-group/
├── aws-ec2/
├── aws-rds/
├── alb-tg/
└── aws-cloudfront-route53/  
```

---

## 🛡️ Project Architecture Highlights

### 🔹 Networking & Security

- **VPC & Subnets** – Isolated network with public and private subnet tiers
- **Internet Gateway** – Public subnet internet routing; no NAT Gateway
- **Security Groups** – ALB → EC2 → RDS traffic restrictions
- **IAM Role & Instance Profile** – AWS permissions for EC2, including SSM where configured
- **Private Database** – RDS is not exposed directly to the internet

### 🔹 Compute & Traffic

- **Application Load Balancer (ALB)** – Routes web traffic to the EC2 target
- **Single EC2 Instance** – Hosts the application in a public subnet
- **Target Group** – Health checks and EC2 registration

### 🔹 Database

- **Amazon RDS** – One DB instance in a private subnet group spanning two AZs
- **Database Access** – RDS security group allows inbound database traffic only from EC2's security group

### 🔹 DNS & Content Delivery 

- **Amazon Route 53** – DNS management for a domain
- **Amazon CloudFront** – CDN and content caching

---

## 💰 Cost Considerations

This project removes NAT Gateways, Elastic IPs, Auto Scaling Groups, and WAF to reduce cost and complexity.

However, **the remaining resources are not necessarily free**:

- EC2 compute and its public IPv4 address
- Application Load Balancer (hourly and usage charges)
- RDS instance and storage
- Route 53 hosted zone
- CloudFront requests and data transfer
- Data transfer and other AWS usage

Check the AWS Pricing Calculator, Free Tier eligibility, and account credits. Set up an AWS Budget and billing alerts before applying.

---

## 🔐 Security Best Practices

- Restrict SSH (port 22) to your own IP, or use Systems Manager Session Manager.
- Allow ALB web traffic only on the intended ports (80/443).
- Allow EC2 application traffic only from the ALB security group where possible.
- Allow RDS database traffic only from the EC2 security group.
- Keep RDS private and disable public accessibility.
- Use IAM roles instead of storing AWS access keys on EC2.
- Store secrets securely and never commit passwords or state files.
- Enable HTTPS with ACM when using a domain.
- Review Terraform plans and AWS billing regularly.


