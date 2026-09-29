# trendmart-aws-infra[README.md](https://github.com/user-attachments/files/32792611/README.md)
# TrendMart Cloud Infrastructure — AWS Hands-On Project

A real-world infrastructure scenario built on AWS (Mumbai region — `ap-south-1`) to practice production-style cloud architecture: secure networking, high availability, and a managed database — starting from a broken single-server setup.

## Problem Statement

> TrendMart is a startup e-commerce app running on a single EC2 instance in the default VPC, using root credentials, with no monitoring, no auto-scaling, and a database running locally on the same server. Goal: rebuild this as secure, scalable, production-ready infrastructure.

## Architecture

```
                         Internet
                            |
                    [Internet Gateway]
                            |
                 ┌──────────────────────┐
                 │   Application LB      │  (Public Subnets)
                 └──────────┬───────────┘
                            |
        ┌───────────────────┴───────────────────┐
        │            Auto Scaling Group           │  (Private Subnets)
        │        EC2 (t2.micro) x 2–4 instances   │
        └───────────────────┬───────────────────┘
                            |
                     [RDS PostgreSQL]              (Private Subnets)
                     (Multi-AZ subnet group)
```

**VPC:** `10.0.0.0/16`
| Subnet | CIDR | AZ | Type |
|---|---|---|---|
| Public-1 | 10.0.1.0/24 | ap-south-1a | Public |
| Public-2 | 10.0.2.0/24 | ap-south-1b | Public |
| Private-1 | 10.0.11.0/24 | ap-south-1a | Private |
| Private-2 | 10.0.12.0/24 | ap-south-1b | Private |

## Tech Stack

- **Compute:** EC2 (Amazon Linux 2023, t2.micro) + Auto Scaling Group
- **Networking:** Custom VPC, public/private subnets, Internet Gateway, route tables
- **Load Balancing:** Application Load Balancer + Target Groups
- **Database:** RDS (PostgreSQL), private subnet, no public access
- **Security:** Layered Security Groups (ALB → EC2 → RDS, least-privilege access)

## Phases Completed

### ✅ Phase 1 — Secure Network Foundation
- Custom VPC with 2 public + 2 private subnets across 2 AZs
- Internet Gateway attached, public route table configured
- Auto-assign public IP enabled on public subnets

### ✅ Phase 2 — Highly Available Compute
- Layered Security Groups: ALB-SG (public HTTP) → EC2-SG (only from ALB) → RDS-SG (only from EC2)
- Launch Template with user-data bootstrap script (Apache web server)
- Application Load Balancer across both public subnets
- Auto Scaling Group (min 2 / desired 2 / max 4) in private subnets, target-tracking on CPU

### ✅ Phase 3 — Managed Database
- RDS PostgreSQL instance in private subnets (no public access)
- Dedicated DB subnet group across 2 AZs
- Access restricted to EC2-SG only

## Screenshots

*(Add screenshots here before pushing — VPC console, working ALB DNS page, ASG instances, RDS "Available" status)*

## Security Highlights

- No component is directly exposed except the Load Balancer
- Database has zero internet access — reachable only from app-tier
- No root/long-lived credentials used for resource access

## Cost Management

This was a **learning exercise, not a production deployment**. All billable resources (EC2, RDS, NAT Gateway if created) were deleted after the exercise to avoid ongoing charges. Only free, static resources (VPC, subnets, route tables, security groups) were left as-is.

## Next Steps (Planned, Not Yet Implemented)

- [ ] CloudWatch — metrics dashboards + alarms (CPU, health checks)
- [ ] Route 53 — custom domain routing to the ALB
- [ ] Lambda — automated backup/alert function
- [ ] Terraform — rebuild this entire setup as Infrastructure as Code

## Lessons Learned

*(Fill in after wrapping up — e.g. NAT Gateway cost tradeoffs, private subnet internet access, security group chaining)*
