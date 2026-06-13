# 🌿 Spring PetClinic — 3-Tier AWS Deployment

> A production-grade deployment of the **Spring PetClinic** application on AWS using a highly available, secure, and scalable **3-Tier Architecture** following AWS Well-Architected Framework best practices.

🔗 **Live Application:** [https://petclinic.eshwar.site](https://petclinic.eshwar.site)

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture Diagram](#-architecture-diagram)
- [AWS Services Used](#️-aws-services-used)
- [Network Design](#-network-design)
- [Security Architecture](#-security-architecture)
- [Load Balancers](#️-load-balancers)
- [Auto Scaling Groups](#-auto-scaling-groups)
- [EC2 Instances](#-ec2-instances)
- [RDS MySQL Database](#️-rds-mysql-database)
- [SSL Certificate](#-ssl-certificate)
- [Monitoring & Observability](#-monitoring--observability)
- [Systems Manager](#-systems-manager)
- [Screenshots](#-screenshots)
- [Deployment Guide](#-deployment-guide)
- [Learning Outcomes](#-learning-outcomes)
- [Future Improvements](#-future-improvements)
- [Author](#-author)

---

## 🎯 Project Overview

This project demonstrates how to deploy a **Java Spring Boot application** on AWS using a clean **3-Tier Architecture** that separates concerns across three distinct layers:

| Layer | Technology | Subnet |
|-------|-----------|--------|
| **Presentation (Frontend)** | React + Nginx | Public Subnets |
| **Application (Backend)** | Spring Boot + Java 17 | Private App Subnets |
| **Database** | Amazon RDS MySQL | Private DB Subnets |

### Key Features

✅ **High Availability** — Multi-AZ deployment across `us-east-1a` and `us-east-1b`  
✅ **Auto Scaling** — Dynamic capacity management based on demand  
✅ **Security** — Layered security groups, SSL/TLS, private subnets  
✅ **Monitoring** — CloudWatch dashboards and real-time metrics  
✅ **Load Balancing** — Public and Internal ALBs for traffic distribution  
✅ **Managed Access** — No SSH keys needed; AWS Systems Manager for instance management  

---

## 🏗️ Architecture Diagram

```
                        ┌─────────────────────────────────┐
                        │        USER / INTERNET          │
                        └────────────────┬────────────────┘
                                         │
                                         ▼
                        ┌─────────────────────────────────┐
                        │     GoDaddy DNS (Custom Domain) │
                        │       petclinic.eshwar.site     │
                        └────────────────┬────────────────┘
                                         │
                                         ▼
                        ┌─────────────────────────────────┐
                        │  ACM SSL Certificate            │
                        │  *.eshwar.site  (HTTPS 443)     │
                        └────────────────┬────────────────┘
                                         │
                ╔════════════════════════╪════════════════════════╗
                ║     PUBLIC ALB  (Internet-Facing)              ║
                ║         Port: 80 → redirect 443                ║
                ╚═══════════════╤════════════════╤══════════════╝
                                │                │
              ┌─────────────────┘                └──────────────────┐
              │                                                     │
   ┌──────────▼──────────┐  us-east-1a     ┌──────────▼──────────┐  us-east-1b
   │   web-sub-a          │                 │   web-sub-b          │
   │   10.0.1.0/24        │                 │   10.0.2.0/24        │
   │  ┌───────────────┐   │                 │  ┌───────────────┐   │
   │  │  Frontend EC2  │   │                 │  │  Frontend EC2  │   │
   │  │  React + Nginx │   │                 │  │  React + Nginx │   │
   │  │  (ASG: 1-2)   │   │                 │  │  (ASG: 1-2)   │   │
   │  └───────────────┘   │                 │  └───────────────┘   │
   └─────────────────────-┘                 └──────────────────────┘
              │                                        │
              └─────────────────┬──────────────────────┘
                                │
                ╔════════════════╪════════════════════════╗
                ║   INTERNAL ALB  (Private)              ║
                ║         Port: 8080                     ║
                ╚═══════════════╤════════════════╤══════╝
                                │                │
              ┌─────────────────┘                └──────────────────┐
              │                                                     │
   ┌──────────▼──────────┐  us-east-1a     ┌──────────▼──────────┐  us-east-1b
   │  private-app-a       │                 │  private-app-b       │
   │  10.0.3.0/24         │                 │  10.0.4.0/24         │
   │  ┌───────────────┐   │                 │  ┌───────────────┐   │
   │  │  Backend EC2   │   │                 │  │  Backend EC2   │   │
   │  │  Spring Boot   │   │                 │  │  Spring Boot   │   │
   │  │  Java 17       │   │                 │  │  Java 17       │   │
   │  │  (ASG: 1-2)   │   │                 │  │  (ASG: 1-2)   │   │
   │  └───────────────┘   │                 │  └───────────────┘   │
   └──────────────────────┘                 └──────────────────────┘
              │                                        │
              └─────────────────┬──────────────────────┘
                                │
                                ▼
         ┌──────────────────────────────────────────────────┐
         │                AMAZON RDS MYSQL                   │
         │              Port: 3306 (Private)                 │
         │  ┌───────────────────┐  ┌───────────────────┐    │
         │  │  private-db-a     │  │  private-db-b     │    │
         │  │  10.0.5.0/24      │  │  10.0.6.0/24      │    │
         │  │  us-east-1a       │  │  us-east-1b       │    │
         │  └───────────────────┘  └───────────────────┘    │
         └──────────────────────────────────────────────────┘
```

### Request Flow

```
User
  │
  ▼
GoDaddy Domain (petclinic.eshwar.site)
  │
  ▼
ACM SSL Certificate (HTTPS Termination)
  │
  ▼
Public ALB → Frontend ASG (React + Nginx)
  │
  ▼
Internal ALB → Backend ASG (Spring Boot / Port 8080)
  │
  ▼
Amazon RDS MySQL (Port 3306)
```

---

## ☁️ AWS Services Used

| Service | Purpose | Configuration |
|---------|---------|---------------|
| **Amazon EC2** | Application Hosting | t3.micro instances |
| **Auto Scaling Groups** | High Availability & Elasticity | Min: 1 · Max: 2 per tier |
| **Application Load Balancer** | Traffic Distribution | Public + Internal ALBs |
| **Amazon RDS MySQL** | Relational Database | db.t3.micro, Multi-AZ |
| **Amazon VPC** | Network Isolation | CIDR: 10.0.0.0/16 |
| **Route Tables** | Network Routing | Public & Private routes |
| **Security Groups** | Layered Network Security | Per-tier access control |
| **Internet Gateway** | Public Internet Access | Attached to VPC |
| **NAT Gateway** | Private Subnet Outbound Access | In public subnet |
| **ACM** | SSL/TLS Certificate | *.eshwar.site (wildcard) |
| **Systems Manager (SSM)** | Instance Management | Fleet Manager, Session Manager |
| **CloudWatch** | Monitoring & Alerting | Dashboards, metrics, alarms |
| **GoDaddy DNS** | Custom Domain Resolution | CNAME → ALB DNS |

---

## 🌐 Network Design

### VPC & Subnet Configuration

| Subnet Name | CIDR | Type | Purpose | AZ |
|-------------|------|------|---------|-----|
| `web-sub-a` | 10.0.1.0/24 | Public | Frontend instances | us-east-1a |
| `web-sub-b` | 10.0.2.0/24 | Public | Frontend instances | us-east-1b |
| `private-app-a` | 10.0.3.0/24 | Private | Backend instances | us-east-1a |
| `private-app-b` | 10.0.4.0/24 | Private | Backend instances | us-east-1b |
| `private-db-a` | 10.0.5.0/24 | Private | RDS MySQL | us-east-1a |
| `private-db-b` | 10.0.6.0/24 | Private | RDS MySQL | us-east-1b |

### VPC Subnets

![VPC Subnets](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/subnets.png)
*Public and private subnets across two availability zones*

### VPC Resource Map

![VPC Resource Map](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/vpc-resource-map.png)
*Complete VPC topology showing all resources and connectivity*

---

## 🔒 Security Architecture

Security is implemented in layers — each tier only accepts traffic from the tier directly above it.

```
Internet (0.0.0.0/0)
  │  Port 80, 443
  ▼
Web Security Group  (Public ALB + Frontend)
  │  Port 8080
  ▼
Application Security Group  (Backend EC2)
  │  Port 3306
  ▼
Database Security Group  (RDS MySQL)
```

| Security Group | Allowed Inbound | Source |
|----------------|----------------|--------|
| **Web SG** | HTTP (80), HTTPS (443) | Internet (0.0.0.0/0) |
| **Application SG** | HTTP (8080), SSH (22) | Web Security Group |
| **Database SG** | MySQL (3306) | Application Security Group |

### Security Highlights

✅ No direct internet access to backend or database  
✅ Backend EC2 instances deployed in **private subnets**  
✅ RDS deployed in **private DB subnets** with no public endpoint  
✅ SSL/TLS enabled via **ACM wildcard certificate**  
✅ **No SSH keys required** — access via AWS SSM Session Manager  
✅ Three-tier security group chain enforces least-privilege  

---

## ⚖️ Load Balancers

### Public ALB (Internet-Facing)

Receives all inbound traffic from the internet, terminates SSL, and routes to the Frontend Auto Scaling Group.

- **Scheme:** Internet-facing
- **Listeners:** HTTP (80) → redirect to HTTPS; HTTPS (443) → forward to frontend target group
- **Multi-AZ:** Yes (us-east-1a, us-east-1b)
- **SSL Certificate:** ACM `*.eshwar.site`

![Public ALB](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/public-alb.png)
*Internet-facing load balancer distributing traffic across frontend instances*

---

### Internal ALB (Private)

Routes traffic from frontend Nginx instances to backend Spring Boot services. Never exposed to the internet.

- **Scheme:** Internal (private subnets only)
- **Listeners:** HTTP (8080) → forward to backend target group
- **Multi-AZ:** Yes (us-east-1a, us-east-1b)

![Internal ALB](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/internal-alb.png)
*Private internal load balancer routing traffic from frontend to backend services*

---

### Target Groups

**Frontend Target Group** — Health check on port 80

![Frontend Target Group](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/frontend-target-group.png)
*Frontend target group with health checks for Nginx instances*

**Backend Target Group** — Health check on port 8080

![Backend Target Group](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/backend-target-group.png)
*Backend target group with health checks for Spring Boot instances*

---

## 📈 Auto Scaling Groups

Both tiers use Auto Scaling Groups with Launch Templates for dynamic capacity management.

| ASG | Min | Desired | Max | Technology |
|-----|-----|---------|-----|-----------|
| **Frontend ASG** | 1 | 1 | 2 | React + Nginx |
| **Backend ASG** | 1 | 1 | 2 | Spring Boot + Java 17 |

### Frontend ASG

![Frontend ASG](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/frontend-asg.png)
*Auto Scaling Group managing frontend (React + Nginx) instances across AZs*

### Backend ASG

![Backend ASG](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/backend-asg.png)
*Auto Scaling Group managing backend (Spring Boot) instances across AZs*

---

## 🖥️ EC2 Instances

### Running Instances

![EC2 Instances](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/ec2.png)
*EC2 instances for frontend and backend tiers running across availability zones*

| Role | Technology | Subnet | Instance Type |
|------|-----------|--------|--------------|
| Frontend | React + Nginx | Public (web-sub-a/b) | t3.micro |
| Backend | Spring Boot + Java 17 | Private (private-app-a/b) | t3.micro |

### Systemd Service (Auto-start on boot)

![Systemd Service File](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/systemd_file.png)
*Systemd service configuration for Spring Boot application auto-startup on boot*

---

## 🗄️ RDS MySQL Database

Amazon RDS MySQL runs in private DB subnets with no public accessibility.

| Setting | Value |
|---------|-------|
| **Engine** | MySQL Community |
| **Instance Type** | db.t3.micro |
| **Storage** | 20 GB gp2 |
| **Subnets** | private-db-a, private-db-b |
| **Public Access** | Disabled |
| **Backup Retention** | 7 days |

![RDS MySQL](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/rds-mysql.png)
*Amazon RDS MySQL instance deployed in private subnets with automated backups*

---

## 🔐 SSL Certificate

A wildcard ACM certificate covers all subdomains of `eshwar.site`, providing HTTPS for the live application.

| Field | Value |
|-------|-------|
| **Certificate** | `*.eshwar.site` |
| **Status** | Issued ✅ |
| **Validation** | DNS |
| **Attached To** | Public ALB — HTTPS Listener |

![ACM Certificate](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/acm-certificate.png)
*AWS Certificate Manager wildcard certificate for *.eshwar.site — status: Issued*

### Domain Routing (GoDaddy)

![Domain Routing](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/domain-routing.png)
*GoDaddy DNS CNAME record pointing petclinic.eshwar.site to the Public ALB DNS name*

---

## 📊 Monitoring & Observability

Amazon CloudWatch provides centralized monitoring across all layers.

**Metrics Tracked:**
- EC2: CPU Utilization, Network In/Out, Status Checks
- ALB: Request Count, Target Response Time, HTTP 4xx/5xx error rates
- RDS: CPU, DB Connections, Freeable Memory, Read/Write IOPS
- ASG: Desired/In-Service capacity, Scaling Activity

![CloudWatch Dashboard](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/cloudwatch-dashboard.png)
*CloudWatch dashboard monitoring EC2, ALB, RDS, and Auto Scaling metrics in real time*

---

## 🛠️ Systems Manager

AWS Systems Manager Fleet Manager provides secure, keyless access to all EC2 instances — no bastion host or SSH keys required.

**Features used:**
- **Fleet Manager** — Inventory and status of all instances
- **Session Manager** — Secure shell-like access without opening port 22
- **Run Command** — Remote command execution at scale

![SSM Fleet Manager](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/ssm-fleet-manager.png)
*AWS Systems Manager Fleet Manager showing all managed EC2 instances*

---

## 📸 Screenshots

### Application Running Live

![Application Running](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/application-working.png)
*Spring PetClinic application successfully deployed and accessible at petclinic.eshwar.site*

### Summary Gallery

| Component | Screenshot |
|-----------|-----------|
| Public ALB | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/public-alb.png) |
| Internal ALB | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/internal-alb.png) |
| Frontend ASG | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/frontend-asg.png) |
| Backend ASG | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/backend-asg.png) |
| Frontend Target Group | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/frontend-target-group.png) |
| Backend Target Group | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/backend-target-group.png) |
| EC2 Instances | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/ec2.png) |
| VPC Subnets | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/subnets.png) |
| VPC Resource Map | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/vpc-resource-map.png) |
| RDS MySQL | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/rds-mysql.png) |
| ACM Certificate | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/acm-certificate.png) |
| Domain Routing | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/domain-routing.png) |
| CloudWatch Dashboard | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/cloudwatch-dashboard.png) |
| SSM Fleet Manager | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/ssm-fleet-manager.png) |
| Systemd Service | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/systemd_file.png) |
| Application Live | ![](https://raw.githubusercontent.com/eeshwardevops/spring-petclinic-3tier-aws-deployment/main/screenshots/application-working.png) |

---

## 🚀 Deployment Guide

### Prerequisites

- AWS CLI configured (`aws configure`)
- Java 17 and Maven installed
- An AWS account with appropriate permissions

### 1. Clone Repository

```bash
git clone https://github.com/eeshwardevops/spring-petclinic-3tier-aws-deployment.git
cd spring-petclinic-3tier-aws-deployment
```

### 2. Build Application

```bash
mvn clean package -DskipTests
# JAR output: target/spring-petclinic-*.jar
```

### 3. Configure Database Connection

Update `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://<RDS-ENDPOINT>:3306/petclinic
spring.datasource.username=<DB_USERNAME>
spring.datasource.password=<DB_PASSWORD>
```

### 4. Deploy Backend

```bash
java -jar target/spring-petclinic-*.jar
```

Or using the systemd service for persistent background execution:

```bash
sudo systemctl enable petclinic
sudo systemctl start petclinic
sudo systemctl status petclinic
```

### 5. Configure Load Balancers

```bash
# Create Target Groups
aws elbv2 create-target-group \
  --name frontend-targets --protocol HTTP --port 80 --vpc-id <VPC_ID>

aws elbv2 create-target-group \
  --name backend-targets --protocol HTTP --port 8080 --vpc-id <VPC_ID>

# Create Public ALB
aws elbv2 create-load-balancer \
  --name public-alb \
  --subnets <web-sub-a> <web-sub-b> \
  --security-groups <web-sg-id> \
  --scheme internet-facing

# Create Internal ALB
aws elbv2 create-load-balancer \
  --name internal-alb \
  --subnets <private-app-a> <private-app-b> \
  --security-groups <app-sg-id> \
  --scheme internal
```

### 6. Configure Auto Scaling

```bash
# Create Launch Templates, then ASGs
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name frontend-asg \
  --launch-template LaunchTemplateName=frontend-lt \
  --min-size 1 --max-size 2 --desired-capacity 1 \
  --vpc-zone-identifier "<web-sub-a>,<web-sub-b>" \
  --target-group-arns <frontend-tg-arn>

aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name backend-asg \
  --launch-template LaunchTemplateName=backend-lt \
  --min-size 1 --max-size 2 --desired-capacity 1 \
  --vpc-zone-identifier "<private-app-a>,<private-app-b>" \
  --target-group-arns <backend-tg-arn>
```

---

## 🎯 Learning Outcomes

Through this project I gained hands-on experience with:

| Domain | Skills |
|--------|--------|
| **AWS Networking** | VPC design, subnets, route tables, IGW, NAT Gateway |
| **Compute** | EC2 administration, AMIs, instance profiles |
| **Load Balancing** | Public & Internal ALBs, target groups, health checks |
| **Auto Scaling** | Launch templates, ASG policies, scaling activities |
| **Database** | RDS MySQL, subnet groups, backup strategies |
| **Security** | Security groups, ACM, SSL/TLS, least-privilege |
| **Monitoring** | CloudWatch dashboards, metrics, alarms, logs |
| **Instance Management** | SSM Fleet Manager, Session Manager, Run Command |
| **Architecture** | High availability, multi-AZ, 3-tier patterns |
| **Production Ops** | Systemd services, zero-downtime considerations |

---

## 🔮 Future Improvements

### Infrastructure & CI/CD
- 🚀 **Terraform IaC** — Replace manual provisioning with code
- 🚀 **GitHub Actions CI/CD** — Automated test → build → deploy pipeline
- 🚀 **Docker + Amazon ECS/Fargate** — Containerized workloads
- 🚀 **Blue/Green Deployments** — Zero-downtime release strategy

### Security & Compliance
- 🔐 **AWS WAF** — Web Application Firewall
- 🔐 **AWS Secrets Manager** — Secure credential storage
- 🔐 **VPC Flow Logs + CloudTrail** — Full audit trail
- 🔐 **AWS Shield** — DDoS protection

### Networking & Performance
- 🌍 **Amazon Route 53** — DNS failover and health-based routing
- 🌍 **Amazon CloudFront** — CDN for static assets
- 🌍 **VPC Endpoints** — Private access to AWS services, reduce NAT costs

### Observability
- 📊 **AWS X-Ray** — Distributed tracing across tiers
- 📊 **Prometheus + Grafana** — Custom application metrics

---

## 👨‍💻 Author

**Eshwar Gajula**

- 🐙 **GitHub:** [@eeshwardevops](https://github.com/eeshwardevops)
- 💼 **LinkedIn:** [Connect on LinkedIn](https://linkedin.com)
- 🌐 **Live Project:** [petclinic.eshwar.site](https://petclinic.eshwar.site)

---

## ⭐ Support

If you found this project helpful, please consider giving it a ⭐ on GitHub — it helps others discover the project!

**Repository:** [spring-petclinic-3tier-aws-deployment](https://github.com/eeshwardevops/spring-petclinic-3tier-aws-deployment)

---

## 📜 License

This project is open-source and available under the [MIT License](LICENSE).

---

> **Last Updated:** June 2026 · **Status:** ✅ Production Ready · **Region:** us-east-1
