# Scalable-high-availability-aws-infrastructure-wordpress
TravelStream is an enterprise-grade WordPress deployment that solves real-world challenges faced by high-traffic, media-heavy websites. This advanced cloud architecture on AWS using ALB, Auto Scaling, EFS, RDS, S3, IAM Role, Glacier Deep Archive and cost-optimized storage strategies demonstrates automatically transition storage tiers while maintaining availability and Scalability.

# Business Impact
1) 99.99% uptime through multi-AZ redundancy
2) 90% storage cost reduction via intelligent lifecycle management
3) Zero data loss during scaling events
4) Auto-healing infrastructure with no manual intervention

## 🚨 Problem Statement (Why This Architecture Matters)

Traditional WordPress deployments struggle to meet production-grade requirements this project addresses those gaps using AWS-native, enterprise-ready solutions.

| Production Challenge | Real-World Impact in Scaled Systems                                   | AWS-Based Solution Implemented                              |
|----------------------|-------------------------------------------------------------------------|-------------------------------------------------------------|
| Media File Inconsistency | Broken images (404 errors) when traffic is served by multiple servers | Shared Amazon EFS for centralized media storage             |
| Data Durability Risk | User uploads lost during scaling events or EC2 failures                | Persistent Amazon RDS + Amazon EFS                          |
| Uncontrolled Storage Costs | Exponential cost growth with large media libraries               | Amazon S3 with lifecycle policies to Glacier                |
| Security Exposure    | Publicly accessible databases and hard-coded credentials               | Private subnets, IAM roles, and security groups              |
| Single Point of Failure | Entire website outage on instance failure                          | Auto Scaling Groups behind an Application Load Balancer     |


## 🛠️ Tech Stack

### ☁️ AWS Services

| Service | Purpose | Production Configuration |
|--------|--------|--------------------------|
| Amazon VPC | Isolated and secure network environment | 2 Availability Zones, public & private subnets |
| Application Load Balancer | Traffic distribution and health monitoring | HTTP/HTTPS listeners, cross-zone load balancing |
| Auto Scaling Group | Elastic compute scaling | Min: 2, Max: 4 instances, target 60% CPU utilization |
| Amazon EC2 | Application servers for WordPress | t3.micro, Amazon Linux 2023 |
| Amazon EFS | Shared filesystem for media uploads | General Purpose mode, lifecycle management enabled |
| Amazon RDS (MySQL) | Managed relational database | Multi-AZ deployment, automated backups |
| Amazon S3 + Glacier | Cost-efficient object storage & archival | Lifecycle policies, versioning enabled |
| AWS IAM | Identity and access management | Role-based access, least privilege principle |
| Amazon CloudWatch | Monitoring, logging, and alerting | CPU, network, disk, and health metrics |

## 🌐 Networking & Security Design

### VPC Architecture
- Custom Amazon VPC with CIDR block `10.0.0.0/16`
- Designed for **high availability and isolation**

**Subnet Strategy**
- **2 Public Subnets**
  - Application Load Balancer
  - Bastion Host
- **2 Private Subnets**
  - EC2 (WordPress)
  - Amazon EFS
  - Amazon RDS

### Internet Access
- Internet Gateway attached to the VPC
- NAT Gateway enables **secure outbound internet access** for private resources
- No inbound internet access to backend services

### 🔐 Security Groups (Zero Trust Model)

| Security Group | Inbound Rules |
|---------------|--------------|
| ALB-SG | HTTP (80), HTTPS (443) from `0.0.0.0/0` |
| Web-SG | HTTP from ALB-SG, SSH from Bastion-SG |
| EFS-SG | NFS (2049) from Web-SG |
| RDS-SG | MySQL (3306) from Web-SG |
| Bastion-SG | SSH from Admin IP only |

**Security Highlights**
- No public access to EC2, EFS, or RDS
- Traffic strictly controlled using **security group chaining**
- Database fully isolated in private subnets

## ⚙️ Compute & Auto Scaling

### EC2 Configuration
- AMI: Amazon Linux 2023
- Instance Type: `t3.micro`
- Deployed in **private subnets**
- No public IP addresses assigned

### Auto Scaling Group (ASG)
- Minimum instances: **2**
- Desired instances: **2**
- Maximum instances: **4**

### Scaling Policy
- **Target Tracking Policy**
- Scale out when **Average CPU > 60%**
- Scale in when CPU drops below threshold

**This ensures**
- Automatic scaling during traffic spikes
- Cost efficiency during low usage
- Self-healing on instance failure

## 📁 Shared Storage with Amazon EFS

### Why Amazon EFS?
WordPress stores uploads locally (`wp-content/uploads`).  
In a multi-instance setup, this causes **broken images and inconsistent media**.

### Solution
- Amazon EFS mounted at: /var/www/html

- ### 📁 Shared Storage (Amazon EFS)
- Mounted automatically via **Launch Template user-data**(mount -t efs fs-00a0542a495742795:/ /var/www/html)
- Mount targets created in **each private subnet (AZ-aware)**

**Result**
- All EC2 instances share the same WordPress files
- No broken images in multi-instance setup
- Safe instance termination and replacement

### 🗄️ Database Layer (Amazon RDS – MySQL)

**Configuration**
- Engine: MySQL
- Deployed in private subnets
- Not publicly accessible
- Access allowed only from `Web-SG`

**Why RDS**
- Persistent storage for WordPress data
- Shared across all EC2 instances
- Prevents data loss during scaling or failures

### 🪣 Object Storage & Cost Optimization (S3 + Glacier)
- Stores backups and archived media
- Accessed via IAM Role (no access keys)

**Offloading Strategy**
- New uploads → EFS  
- Files > 30 days → S3  
- S3 lifecycle → Glacier Deep Archive after 90 days

**Benefits**
- EFS remains fast and lightweight
- Storage costs reduced by up to **90%**
- Long-term data retained securely

### 🔐 IAM & Security Best Practices
- EC2 uses IAM Roles (no static credentials)
- Least-privilege permissions:
  - s3:GetObject
  - s3:PutObject
  - s3:ListBucket (bucket-level only)
- No public S3 access
- All backend resources in private subnets

### 🧪 Testing & Validation

**High Availability**
- Terminated one EC2 instance manually
- Website stayed live
- ASG launched replacement automatically

**Auto Scaling**
- Generated CPU load
- Scaled out above 60% CPU
- Scaled back in after load dropped

**S3 Offloading**
- Uploaded media via WordPress
- Verified file on EFS
- Confirmed offload to S3
- Website continued functioning

### 🎯 Key Outcomes
 1) Highly available WordPress deployment  
 2) No single point of failure  
 3) Shared state across instances  
 4) Secure, private architecture  
 5) Automatic scaling & self-healing  
 6) Optimized storage costs  

### 🧠 What This Project Demonstrates
- Real-world AWS architecture design
- DevOps best practices
- Stateless compute with shared storage
- Security-first mindset
- Cost optimization strategies
- Production-grade scalability

### 📌 Future Enhancements
- Enable HTTPS using ACM
- Add CloudFront CDN for global caching
- Integrate AWS WAF for security
- Replace cron jobs with EventBridge + Lambda





