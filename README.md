#  My Platform Project — Highly Available AWS Infrastructure with Terraform

A production-style, highly available web application infrastructure on **AWS**, provisioned entirely with **Terraform**. The setup includes a custom VPC across two Availability Zones, public/private subnets, a NAT Gateway, a Bastion Host for secure SSH access, an Application Load Balancer, and an Auto Scaling Group of Apache web servers.

![Terraform](https://img.shields.io/badge/Terraform-%3E%3D1.0-844FBA?logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?logo=amazonaws&logoColor=white)
![Provider](https://img.shields.io/badge/aws--provider-~%3E6.0-blue)
![Status](https://img.shields.io/badge/status-active-brightgreen)

---

##  Architecture Diagram

![Architecture Diagram](architecture-diagram.svg)

**Traffic flow:** Internet → Internet Gateway → Application Load Balancer (public subnets) → Auto Scaling Group of EC2 instances (private subnets) → Apache web server. Outbound internet access for the private instances (e.g. `apt update`) is routed through the NAT Gateway. Admin SSH access to private instances goes through the Bastion Host in the public subnet.

---

##  Infrastructure Components

| Component | Description |
|---|---|
| **VPC** | `10.0.0.0/16` custom VPC (`demo-vpc`) |
| **Subnets** | 2 public + 2 private subnets across `us-east-1a` and `us-east-1b` |
| **Internet Gateway** | Provides internet access to public subnets |
| **NAT Gateway** | Allows private subnet instances outbound internet access |
| **Route Tables** | Separate public and private route tables with associations |
| **Security Groups** | Dedicated SGs for ALB, Bastion Host, and private EC2 instances |
| **Bastion Host** | EC2 instance in the public subnet for secure SSH jump access |
| **Application Load Balancer** | Distributes HTTP traffic across the Auto Scaling Group |
| **Auto Scaling Group** | Launches EC2 instances (via Launch Template) running Apache |
| **User Data Script** | Installs Apache and serves a dynamic HTML page showing hostname & private IP |

---

##  Project Structure

```
.
├── terraform-provider.tf                  # AWS provider & required version
├── terrraform-variable.tf                 # All input variables
├── terraform-vpc.tf                       # VPC resource
├── terraform-subnets.tf                   # Public & private subnets
├── terraform-internetgateway.tf           # Internet Gateway
├── terraform-NAT-gateway.tf               # NAT Gateway + Elastic IP
├── terraform-route-tables.tf              # Public & private route tables
├── terraform-asossition-route-table.tf    # Route table associations
├── terraform-SG.tf                        # Security groups (ALB, Bastion, Private EC2)
├── terrsform-Bastion-EC2.tf               # Bastion host EC2 instance
├── terraform-ALB.tf                       # Application Load Balancer + Target Group + Listener
├── terraform-ASG.tf                       # Launch Template + Auto Scaling Group
├── terraform-user-data.sh                 # Bootstrap script (Apache installation)
├── terraform-output.tf                    # Outputs
├── terraform.tfstate / .backup            # Terraform state files
└── README.md
```

---

##  Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/downloads) `>= 1.0`
- An AWS account with configured credentials (`aws configure` or environment variables)
- An existing EC2 Key Pair in `us-east-1` matching the `key_name` variable (default: `demo-key`)
- AWS provider version `~> 6.0`

---

##  Configuration (Variables)

Key variables (defined in `terrraform-variable.tf`) you may want to review or override:

| Variable | Default | Description |
|---|---|---|
| `vpc_cidr` | `10.0.0.0/16` | VPC CIDR block |
| `demo_public_subnet_1_cidr` / `_2_cidr` | `10.0.1.0/24` / `10.0.3.0/24` | Public subnet CIDRs |
| `demo_private_subnet_1_cidr` / `_2_cidr` | `10.0.2.0/24` / `10.0.4.0/24` | Private subnet CIDRs |
| `ami_id` | `ami-0b6d9d3d33ba97d99` | AMI used for Bastion & ASG instances |
| `instance_type` | `t3.micro` | EC2 instance type |
| `key_name` | `demo-key` | EC2 key pair name |
| `asg_min_size` / `asg_max_size` / `asg_desired_capacity` | `1` / `3` / `2` | Auto Scaling Group sizing |

---

##  Deployment Steps

```bash
# 1. Clone / navigate into the project folder
cd my-platform-project

# 2. Initialize Terraform (downloads the AWS provider)
terraform init

# 3. Validate the configuration
terraform validate

# 4. Review the execution plan
terraform plan

# 5. Apply and provision the infrastructure
terraform apply

# 6. When done, tear everything down
terraform destroy
```

After `apply` completes, grab the ALB DNS name from the AWS Console (EC2 → Load Balancers) and open it in your browser to see the running web page.

---

##  Accessing the Application

1. Open the AWS Console → **EC2 → Load Balancers → demo-alb**
2. Copy the **DNS name**
3. Paste it into your browser — you should see the **"Welcome to My Platform Project"** page along with the serving instance's hostname and private IP
4. Refresh a few times — the ALB round-robins across healthy instances in the Auto Scaling Group, so the hostname/IP shown will change

To SSH into a private instance:

```bash
# 1. SSH into the Bastion Host first
ssh -i demo-key.pem ubuntu@<bastion-public-ip>

# 2. From the Bastion, hop into a private instance
ssh -i demo-key.pem ubuntu@<private-instance-ip>
```

---

##  Screenshots

> Add your own deployment screenshots here — this makes the README look polished and proves the infrastructure is actually working. Save your images inside a `screenshots/` folder in the project and reference them as shown below.

| Screenshot | Description |
|---|---|
| ![VPC](screenshots/vpc-resource-map.png) | VPC Resource Map — subnets, route tables, and gateways |
| ![ALB](screenshots/alb-console.png) | Application Load Balancer console showing `demo-alb` active |
| ![ASG](screenshots/asg-instances.png) | Auto Scaling Group with healthy running instances |
| ![Target Group](screenshots/target-group-health.png) | Target Group showing healthy targets |
| ![Website](screenshots/website-output.png) | Browser output — the Apache welcome page with hostname & IP |
| ![Bastion SSH](screenshots/bastion-ssh.png) | Terminal — SSH into Bastion → private instance |

**Suggested screenshots to capture (in order):**
1. `terraform apply` completing successfully in the terminal
2. VPC Resource Map (AWS Console → VPC → Your VPC → Resource Map)
3. Load Balancer + Target Group health status
4. Auto Scaling Group instances list (all `InService`)
5. Browser showing the live website
6. Terminal showing Bastion → private EC2 SSH hop

---

##  Cleanup

Always destroy resources after testing to avoid unwanted AWS charges:

```bash
terraform destroy
```

---

##  Tech Stack

- **IaC:** Terraform (`~> 1.0`), AWS Provider `~> 6.0`
- **Compute:** EC2, Auto Scaling Group, Launch Template
- **Networking:** VPC, Subnets, Internet Gateway, NAT Gateway, Route Tables
- **Load Balancing:** Application Load Balancer + Target Group
- **Web Server:** Apache2 on Ubuntu

---

##  License

This project is for learning/demo purposes. Feel free to fork and adapt it for your own use case.
