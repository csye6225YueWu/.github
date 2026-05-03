# ☁️ Cloud-Native Application Platform

### Backend APIs · Infrastructure as Code · Event-Driven Services

An end-to-end cloud-native application platform built across three repositories — covering backend API development, infrastructure provisioning, and serverless event-driven workflows on AWS.

> **Stack:** Python · FastAPI · PostgreSQL · Terraform · Packer · GitHub Actions · AWS (15+ services)

---

## 🏗️ Architecture

![Cloud Architecture](https://github.com/csye6225YueWu/.github/blob/main/cloud%20architecture.png?raw=1)


The system separates concerns across three layers: a REST API backend, fully Terraform-managed AWS infrastructure, and an SNS-triggered serverless email verification workflow — all connected through automated CI/CD pipelines across dev and demo environments.

---

## ✨ Project Highlights

**Security**
- 4 Customer Managed KMS keys (EC2/EBS, RDS, S3, Secrets Manager) with 90-day auto-rotation
- All credentials fetched at runtime from Secrets Manager via IMDSv2 — nothing baked into AMIs
- Separate least-privilege IAM roles for EC2 and Lambda

**Reliability**
- ALB + ASG with CPU-based scale-up/scale-down policies across multiple availability zones
- Instance Refresh on every deployment — zero-downtime rolling update with 50% minimum healthy
- `GET /healthz` health check validates DB connectivity before ALB routes traffic

**Observability**
- Custom StatsD middleware tracks per-endpoint request count and latency → CloudWatch
- SQLAlchemy event hooks time every DB query (SELECT / INSERT / UPDATE / DELETE); slow queries (>100ms) trigger warnings
- Structured logging with request correlation across all log lines

**CI/CD**
- 4-stage GitHub Actions pipeline: integration tests → artifact build → Packer AMI → demo deploy
- 49 automated API tests (Newman/Postman) run against a live PostgreSQL service container on every PR
- Packer AMI built in dev account and shared to demo account automatically post-merge

**Infrastructure as Code**
- 67 AWS resources provisioned across `main.tf`, `kms.tf`, `secrets.tf`, `acm.tf`, and `serverless.tf`
- Dev / demo environment isolation via Terraform workspaces and separate AWS accounts
- All changes go through PR → CI → `terraform apply` — no console drift

---

## 📂 Repository Breakdown

### 🌐 [cloud-native-webapp](https://github.com/csye6225YueWu/cloud-native-webapp)

REST API backend with authentication, data persistence, file storage, and full observability.

**API surface — 15 routes:**
- `GET /healthz` — liveness check with DB validation
- `POST /v1/user` — user registration (triggers SNS → email verification)
- `GET /v1/user/self`, `PUT /v1/user/self` — authenticated profile management
- `GET | POST | PUT | DELETE /v1/courses` — course CRUD with Basic Auth
- `POST | GET | DELETE /v1/courses/{id}/syllabus` — S3-backed file upload / download / delete
- `GET /validateEmail` — email verification token handler

**Key implementation details:**
- SQLAlchemy ORM with Alembic migrations; Pydantic schemas for request/response validation
- S3 presigned URL generation for syllabus file access
- StatsD middleware + SQLAlchemy event hooks for CloudWatch observability
- Packer HCL2 template builds AMI with Python runtime, app code, and CloudWatch Agent pre-installed

### 🏗️ [terraform-cloud-infra](https://github.com/csye6225YueWu/terraform-cloud-infra)

Terraform configuration for the complete AWS environment — from networking through application delivery.

**67 resources across 5 files — includes:**
- VPC, public/private subnets, Internet Gateway, route tables
- ALB with HTTP→HTTPS redirect, HTTPS listener, Target Group, health checks
- ASG Launch Template, scale-up/scale-down CloudWatch alarms
- RDS PostgreSQL (KMS-encrypted, private subnet)
- S3 bucket (KMS-encrypted, lifecycle policy, public access blocked)
- IAM roles for EC2 (S3 + CloudWatch + Secrets Manager) and Lambda
- Route 53 A record, ACM certificate with DNS validation
- SNS topic, Lambda function, DynamoDB table
- Secrets Manager secrets for DB credentials and email config (KMS-encrypted)

### ⚡ [cloud-serverless](https://github.com/csye6225YueWu/cloud-serverless)

SNS-triggered Lambda function for asynchronous email verification.

**Workflow:**
1. User registers → API publishes event to SNS
2. SNS triggers Lambda
3. Lambda fetches email config from Secrets Manager
4. Lambda checks DynamoDB to prevent duplicate sends (idempotency by email hash key)
5. Lambda sends verification link via SES
6. User clicks link → `GET /validateEmail` marks account verified

---

## 🛠️ Tech Stack

| Category | Technologies |
|---|---|
| Backend | Python 3.12, FastAPI, SQLAlchemy, Alembic, Pydantic |
| Database | PostgreSQL (RDS), DynamoDB |
| Infrastructure | Terraform, Packer (HCL2) |
| AWS Compute | EC2, Lambda, Auto Scaling Group |
| AWS Networking | VPC, ALB, Route 53, ACM |
| AWS Storage | S3, RDS |
| AWS Security | KMS, Secrets Manager, IAM |
| AWS Messaging | SNS, SES |
| Observability | CloudWatch, StatsD |
| CI/CD | GitHub Actions, Newman (Postman CLI) |

---

## 🚀 Deployment

Two isolated AWS environments — `dev` and `demo` — managed through separate Terraform workspaces and AWS accounts. Every merge to `main` triggers:

1. **Integration tests** — 49 Newman test cases run against a live app + PostgreSQL service container
2. **Artifact build** — application code packaged as a GitHub Actions artifact
3. **AMI build** — Packer builds a new EC2 AMI in dev account; AMI shared to demo account
4. **Instance Refresh** — demo ASG performs rolling update to the new AMI (50% min healthy, 45-min timeout)

Custom domain `demo.yw-csye6225.me` serves HTTPS traffic via TLS 1.3 with a DNS-validated ACM certificate.

---

## 🔗 Repositories

| | |
|---|---|
| 🌐 Backend | [csye6225YueWu/cloud-native-webapp](https://github.com/csye6225YueWu/cloud-native-webapp) |
| 🏗️ Infrastructure | [csye6225YueWu/terraform-cloud-infra](https://github.com/csye6225YueWu/terraform-cloud-infra) |
| ⚡ Serverless | [csye6225YueWu/cloud-serverless](https://github.com/csye6225YueWu/cloud-serverless) |
