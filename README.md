# Spring PetClinic 3-Tier Enterprise AWS Deployment

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?style=for-the-badge&logo=amazon-aws)
![Java](https://img.shields.io/badge/Java-17-blue?style=for-the-badge&logo=openjdk)
![SpringBoot](https://img.shields.io/badge/SpringBoot-3-green?style=for-the-badge&logo=spring-boot)
![React](https://img.shields.io/badge/React-Frontend-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Nginx](https://img.shields.io/badge/Nginx-Reverse_Proxy-009639?style=for-the-badge&logo=nginx)
![MySQL](https://img.shields.io/badge/MySQL-RDS-4479A1?style=for-the-badge&logo=mysql)
![CloudWatch](https://img.shields.io/badge/Monitoring-CloudWatch-912121?style=for-the-badge&logo=amazon-cloudwatch)
![Status](https://img.shields.io/badge/Status-Production_Ready-success?style=for-the-badge)

## 🌐 Live Application
**URL:** [https://petclinic.eshwar.site](https://petclinic.eshwar.site)  
**Status:** Production Ready

---

## 🚀 Project Overview
This repository hosts the infrastructure blueprint, application workflow, and automation scripts used to deploy the **Spring PetClinic application** (React frontend + Spring Boot backend) on AWS. 

The core deployment utilizes a highly resilient, highly available, and strictly isolated **3-Tier Production Architecture** across multiple Availability Zones. Security controls, traffic distribution, zero-single-point-of-failure routing, domain mapping, and operational observability have been engineered natively following the AWS Well-Architected Framework.

---

## 🏗️ AWS 3-Tier Architecture

The system segregates workloads into logical boundaries: **Presentation (Web Tier)**, **Application (App Tier)**, and **Data (DB Tier)** distributed evenly across `us-east-1a` and `us-east-1b` Availability Zones.

![AWS 3-Tier Architecture](./architecture-diagrams/petclinic-aws-architecture.png)

### 🔄 End-to-End Traffic Workflow

1. **DNS Resolution & Encryption:** Users point their browsers to `https://petclinic.eshwar.site`. **GoDaddy DNS** maps names to the external Load Balancer, while **AWS Certificate Manager (ACM)** handles transit-layer SSL/TLS validation.
2. **Web Layer Filtering:** The **Public Application Load Balancer (ALB)** intercepting external requests validates TLS handshakes and forwards standard traffic down to the **Frontend Auto Scaling Group (ASG)**.
3. **Static Web Serving & UI Reverse Proxy:** Inside the public subnets, EC2 instances running **Nginx** serve static React production artifacts. When a React action hits `/api/*`, Nginx acts as a reverse proxy, mapping queries directly to the Internal Application Load Balancer.
4. **App Layer Load Balancing:** The **Internal ALB** coordinates horizontal traffic switching between the private computing spaces across both AZs.
5. **Business Logic Execution:** The **Backend Auto Scaling Group (ASG)** ingests clean API actions, launching and running **Spring Boot (Java 17)** instances that host core backend capabilities.
6. **Isolated Data Store:** The application engines access an active **Amazon RDS MySQL Multi-AZ Deployment** hidden strictly within isolated DB subnets. 

---

## ☁️ AWS Services Catalog
* **Networking:** Amazon VPC, Public & Private Subnets, Route Tables, Internet Gateway, NAT Gateway.
* **Compute & Scale:** Amazon EC2, EC2 Launch Templates, Auto Scaling Groups (ASG).
* **Traffic Management:** Public Application Load Balancer, Internal Application Load Balancer, Target Groups.
* **Security & Identity:** AWS Identity and Access Management (IAM) Roles, Security Groups, AWS Certificate Manager (ACM).
* **Database:** Amazon RDS MySQL (Multi-AZ Infrastructure).
* **Management & Observability:** AWS Systems Manager (SSM) Fleet Manager, Amazon CloudWatch Dashboards & Logs, GoDaddy DNS Integration.

---

## 🌐 Network Topology & Subnet Grid