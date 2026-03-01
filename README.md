# Highly Available Web Application on AWS

## Overview
Designed and deployed a production-grade highly available web application 
on AWS. The app remains online even if an entire Availability Zone fails — 
with zero manual intervention.

## Architecture
```
                    INTERNET
                       │
                       ▼
            ┌─────────────────────┐
            │   Application Load  │
            │   Balancer (ALB)    │
            │   Public-facing     │
            │   Spans 2 AZs       │
            └──────────┬──────────┘
                       │
           ┌───────────┴───────────┐
           │                       │
    ┌──────▼──────┐         ┌──────▼──────┐
    │  AZ-1 (1a)  │         │  AZ-2 (1b)  │
    │Private Subnet│        │Private Subnet│
    │  ┌────────┐ │         │  ┌────────┐ │
    │  │  EC2   │ │         │  │  EC2   │ │
    │  │(Apache)│ │         │  │(Apache)│ │
    │  └────────┘ │         │  └────────┘ │
    └─────────────┘         └─────────────┘
```

## AWS Services Used
- **VPC** — Custom network with public/private subnet isolation
- **EC2** — Apache web servers in private subnets (t3.micro)
- **Application Load Balancer (ALB)** — Distributes traffic across both AZs
- **Auto Scaling Group (ASG)** — Min: 2, Desired: 2, Max: 4 instances
- **NAT Gateway** — Secure outbound internet access for private instances
- **Security Groups** — Layered security: internet → ALB SG → EC2 SG only

## Key Features
- **Multi-AZ deployment** across us-east-1a and us-east-1b
- **Private subnets** for EC2 — instances never directly exposed to internet
- **ALB health checks** automatically remove unhealthy instances
- **ASG auto-recovery** — replaces a failed instance within ~3 minutes
- **Defense in depth** — EC2 only accepts traffic from ALB security group

## How High Availability Was Validated
Terminated a live EC2 instance during active traffic.
Result: Zero downtime. Traffic instantly routed to surviving instance.
ASG automatically launched a replacement within 3 minutes. ✅

## Network Design
| Subnet | AZ | CIDR | Type |
|---|---|---|---|
| public-subnet-az1 | us-east-1a | 10.0.1.0/24 | Public (ALB) |
| public-subnet-az2 | us-east-1b | 10.0.2.0/24 | Public (ALB) |
| private-subnet-az1 | us-east-1a | 10.0.3.0/24 | Private (EC2) |
| private-subnet-az2 | us-east-1b | 10.0.4.0/24 | Private (EC2) |

## What I Learned
- How to design a VPC from scratch with proper network segmentation
- Why security group chaining is critical for layered security
- How ALB health checks and ASG work together for self-healing infrastructure
- The importance of health check grace periods during instance bootstrapping
