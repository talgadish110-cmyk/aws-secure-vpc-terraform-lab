# AWS Infrastructure & Security Hardening Lab (Terraform & CLI Automation)

## 📌 Project Overview
This project documents the deployment, security hardening, and operational validation of a secure multi-tier web application infrastructure on AWS using **Terraform** and **AWS CLI**. The architecture strictly follows cloud security best practices, ensuring private backend isolation and controlled administrative access.

---

## 🏗 Architecture & Core Components
* **Virtual Private Cloud (VPC):** Custom CIDR block (`10.0.0.0/16`) with DNS support enabled.
* **Subnet Segmentation:**
  * **Public Subnets:** Hosting the Application Load Balancer (ALB) and Bastion Host across two Availability Zones (`us-east-1a`, `us-east-1b`).
  * **Private Subnets:** Hosting the backend Auto Scaling Group (ASG) web servers, completely isolated from direct internet access.
* **Connectivity & Routing:**
  * **Internet Gateway (IGW):** Provides public internet access for the VPC.
  * **NAT Gateway & Elastic IP:** Deployed in Public Subnet 1 to allow private backend instances to securely fetch updates without exposing them inbound.
  * **Route Tables:** Configured dedicated public and private route tables with appropriate associations.

---

## 🔒 Security Hardening & Access Control
* **Security Groups (SG):**
  * **Bastion SG:** Restricts inbound SSH (Port 22) to authorized administrative IPs only.
  * **Web/ASG SG:** Completely blocks direct external access; permits inbound HTTP (Port 80) exclusively from the ALB, and SSH strictly from the Bastion Host security group.
* **IAM & S3 Challenge Integration:**
  * Created a dedicated **IAM Role and Instance Profile** (`bastion_s3_upload_role`) attached directly to the Bastion Host.
  * Deployed a secure, private **S3 Bucket** (`lab-web-assets-*`) with `force_destroy` enabled for lab lifecycle management.
  * Configured the architecture so that *only* the Bastion Host possesses the required IAM permissions to interact with and upload assets to the S3 bucket via AWS CLI.

---

## 🛠️ Operational Workflow & CLI Verification
1. **Infrastructure Provisioning:** Automated the entire deployment lifecycle using declarative **Terraform** configurations (`terraform apply`).
2. **Administrative Access:** Connected securely to the Bastion Host via **AWS EC2 Instance Connect / SSH**.
3. **AWS CLI S3 Verification:** 
   * Created and verified local web assets (`index.html`).
   * Validated permissions and successfully executed CLI commands to list and upload files directly to the secure S3 bucket (`aws s3 cp`).

```bash
# Example CLI validation executed from the Bastion Host:
echo "<h1>Hello from Bastion CLI Lab Challenge!</h1>" > index.html
aws s3 ls
aws s3 cp index.html s3://lab-web-assets-<bucket-id>/


🧹 Lifecycle Management & Cost Optimization
To adhere to financial and operational best practices, the entire infrastructure was cleanly torn down post-verification using Terraform to prevent unnecessary cloud resource costs:
terraform destroy
