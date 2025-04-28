# AWS Three-Tier Architecture Deployment (Manual via Console)

## Table of Contents
- [Network Setup](#network-setup)
  - [VPC](#vpc)
  - [Subnets](#subnets)
  - [Internet Gateway](#internet-gateway)
  - [NAT Gateway](#nat-gateway)
  - [Route Tables](#route-tables)
  - [Security Groups](#security-groups)
- [Application Load Balancer](#application-load-balancer)
- [Database (RDS)](#database-rds)
- [Compute (Auto Scaling Groups)](#compute-auto-scaling-groups)
- [Testing](#testing)

---

# Network Setup

## VPC
- Create a VPC with the following settings:
  - Name: `MyVPC`
  - IPv4 CIDR block: `10.0.0.0/16`
  - Default settings for the rest.

![VPC Created](screenshots/vpc-created.png)

## Subnets
Create 6 subnets:
- Public Subnets:
  - `public-subnet-1a` (AZ: ap-southeast-3a, CIDR: 10.0.0.0/24)
  - `public-subnet-1b` (AZ: ap-southeast-3b, CIDR: 10.0.1.0/24)
- Private Subnets:
  - `private-subnet-1` (AZ: ap-southeast-3a, CIDR: 10.0.2.0/24)
  - `private-subnet-2` (AZ: ap-southeast-3b, CIDR: 10.0.3.0/24)
  - `private-subnet-db-1` (AZ: ap-southeast-3a, CIDR: 10.0.4.0/24)
  - `private-subnet-db-2` (AZ: ap-southeast-3b, CIDR: 10.0.5.0/24)

![Subnets Created](screenshots/subnets-created.png)

## Internet Gateway
- Create an Internet Gateway named `MyIGW`.
- Attach `MyIGW` to `MyVPC`.

![Internet Gateway Created](screenshots/igw-created.png)

## NAT Gateway
- Create a NAT Gateway named `MyNAT` in `public-subnet-1a`.
- Allocate a new Elastic IP if needed.

![NAT Gateway Created](screenshots/nat-created.png)

## Route Tables
- Create `public-rtb` and associate it with public subnets.
- Add a route `0.0.0.0/0` to the Internet Gateway (`MyIGW`).
- Rename the default route table to `private-rtb` and associate it with private subnets.
- Add a route `0.0.0.0/0` to the NAT Gateway (`MyNAT`).

![Route Tables Configured](screenshots/route-tables-configured.png)

## Security Groups
Create the following security groups:

- `front-end-alb`: Allow HTTP (port 80) from Anywhere (0.0.0.0/0).
- `front-end-ec2`: Allow HTTP from `front-end-alb` security group.
- `back-end-alb`: Allow HTTP from `front-end-ec2` security group.
- `back-end-ec2`: Allow HTTP from `back-end-alb` security group.
- `rds-sg`: Allow MySQL/Aurora (port 3306) from `back-end-ec2` security group.

![Security Groups Created](screenshots/security-groups-created.png)

---

# Application Load Balancer

## Target Groups
- Create `front-end-tg` (protocol HTTP, path `/index.php`).
- Create `back-end-tg` (protocol HTTP, path `/api.php`).

## Load Balancers
- Create `front-end-alb` (Internet-facing) attached to public subnets.
- Create `back-end-alb` (Internal) attached to private subnets.

![Load Balancers Created](screenshots/alb-created.png)

---

# Database (RDS)

## DB Subnet Group
- Create `db-subnet` with subnets `private-subnet-db-1` and `private-subnet-db-2`.

## RDS Instance
- Create a MySQL database instance:
  - DB instance identifier: `mydb3tier`
  - Username: `admin`
  - Password: `inipassword123#`
  - DB subnet group: `db-subnet`
  - VPC: `MyVPC`

![RDS Created](screenshots/rds-created.png)

---

# Compute (Auto Scaling Groups)

## Launch Templates
- **Front-end Launch Template**:
  - Use custom AMI built from EC2 instance with HTTP server and PHP.
- **Back-end Launch Template**:
  - Use standard Amazon Linux 2023 AMI with User Data script to set up PHP and connect to RDS.

## Auto Scaling Groups
- Create `front-end-asg`:
  - Attach to public subnets.
  - Attach to `front-end-tg`.
  - Desired capacity: 2, Min: 1, Max: 4.

- Create `back-end-asg`:
  - Attach to private subnets.
  - Attach to `back-end-tg`.
  - Desired capacity: 2, Min: 1, Max: 4.

![Auto Scaling Groups Created](screenshots/asg-created.png)

---

# Testing

- Access the DNS name of `front-end-alb` via browser.
- Ensure the front-end can communicate with the back-end and back-end can access the database.

![Application Tested](screenshots/testing-success.png)

---

> **Note:**
> - Update placeholder values such as backend URL or RDS endpoint where necessary.
> - Ensure security group references are properly configured.
> - Confirm that instances are correctly registered to the target groups.

---

**Created by:** Your Name Here  
**Date:** 2025
