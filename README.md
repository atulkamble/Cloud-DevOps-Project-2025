# Cloud-DevOps-Project-2025
# Cloudnautic — AWS DevOps Project Requirements
**Project Title:** Cloudnautic Customer-Facing SaaS — Multi‑Stage Delivery on AWS (Planning → Dev → Staging → Release → Production)  
**Owner:** Cloudnautic | AWS DevOps Training & Consulting  
**Audience:** Interns, Junior DevOps Engineers, Trainees  
**Last Updated:** 2025‑09‑13

---

## 1) Project Statement
Build an end‑to‑end, production‑grade delivery pipeline on AWS for a sample SaaS web application (containerized Python/Flask API + React UI) with **five lifecycle stages**: **Planning, Development, Staging, Release, Production**.  
Pipeline must be **infrastructure‑as‑code**, **Git‑driven**, **security‑first**, and **cost‑aware**, enabling Cloudnautic teams to:
- Ship changes safely via gated promotions (Dev → Staging → Release → Prod).
- Enforce quality (tests, scans, policies) and operational readiness (monitoring, logging, rollback).
- Demonstrate real‑world DevOps practices for Cloudnautic internships and client trainings.

## 2) In‑Scope Components
- **Source Control:** GitHub (mono‑repo with app + IaC).
- **IaC:** Terraform for AWS (VPC, EKS, ECR, RDS or Aurora Serverless v2 optional, S3, CloudFront, Route 53, IAM, KMS, CloudWatch).
- **Containers & Orchestration:** Docker, Amazon ECR, Amazon EKS (with managed node groups or Fargate profile).
- **CI/CD:** GitHub Actions (primary). Jenkins optional (bonus).
- **App:** Example microservice (Flask API) + static UI (React) packaged as containers, deployed via **Helm**.
- **Observability:** CloudWatch (logs/alarms), optional Prometheus + Grafana (add‑on).
- **Security:** OIDC‑based GitHub→AWS auth, IAM least privilege, KMS encryption, ECR scan on push, Trivy scan in CI, S3 bucket policies, TLS (ACM) + HTTPS via ALB/Ingress/NLB as applicable.
- **Environments:** `dev`, `staging`, `release`, `prod`. Planning is process‑only (no infra).
- **Networking:** 3‑tier VPC (public/priv subnets across ≥2 AZs), NAT GW (cost‑aware: 1 NAT GW acceptable in training; in prod prefer 1/AZ).

## 3) Out‑of‑Scope (for internship baseline)
- Complex multi‑region active/active (stretch goal).
- Advanced service mesh (Linkerd/Istio).
- Complex data migrations (use basic schema init).
- Enterprise SSO/SCIM (use IAM/OIDC + GitHub teams mapping).

## 4) Functional Requirements
1. **Build & Test:** PRs trigger CI (lint, unit tests, SCA, container build).
2. **Image Management:** Tag images with commit SHA and semver; push to ECR.
3. **Deployments:** Helm‑based rollout to EKS `dev` on merge to `develop`; manual approval → `staging`; release tagging (e.g., `vX.Y.Z`) promotes to `release`; change request & final approval → `prod`.
4. **Traffic & TLS:** Public HTTPS endpoint via ALB Ingress Controller or NLB + Ingress; ACM certificate for domain.
5. **DB:** Optional RDS (PostgreSQL) for API; secrets via AWS Secrets Manager/SSM.
6. **Config:** Per‑env values via Helm `values-{env}.yaml` and SSM Parameter Store.
7. **Rollback:** Previous Helm release kept and restorable within 1 command.
8. **Monitoring:** Health checks, 4xx/5xx alarms, CPU/Memory thresholds, error rate alarms; synthetic check optional (Canary).
9. **Logging:** Application stdout/stderr → CloudWatch Logs; retain 14–30 days (training) and 90+ in prod.
10. **Cost Controls:** Instance sizes small in training; auto‑shutdown policy for dev (optional).

## 5) Non‑Functional Requirements
- **Reliability:** ≥2 AZs for prod node groups, HPA enabled.
- **Security:** Private subnets for nodes; only Ingress public; least‑privileged IAM Roles for Service Accounts (IRSA).
- **Performance:** P95 latency SLO ≤ 300 ms on staging/production under nominal load (toy workload).
- **Compliance (lightweight):** Tagging: `project=cloudnautic-devops`, `env=<env>`, `owner=cloudnautic`.
- **Documentation:** READMEs, runbooks, diagrams, onboarding checklist for interns.

## 6) Deliverables
- **D1:** Architecture diagram + README.
- **D2:** Terraform IaC with per‑env workspaces or directories.
- **D3:** Helm chart(s) and per‑env values.
- **D4:** GitHub Actions pipelines (CI + CD with approvals).
- **D5:** Runbooks (deploy, rollback, on‑call basics).
- **D6:** Demo recording (5–10 min) showing PR→Prod flow.

## 7) Milestones & Acceptance Criteria
- **M1 (Week 1):** Repo scaffold, Planning artifacts. *AC:* Repo structure & issues created.
- **M2 (Week 2):** VPC + EKS + ECR ready in `dev`. *AC:* `kubectl get nodes` shows ready, push to ECR works.
- **M3 (Week 3):** CI passes, app deploys to `dev`. *AC:* `/healthz` returns 200 via Ingress + HTTPS.
- **M4 (Week 4):** Staging & Release gates, alarms, rollback. *AC:* Manual approvals, alarm fires on forced 5xx.
- **M5 (Week 5):** Production cutover. *AC:* Release tag deploys to `prod`, rollback validated.

## 8) Risks & Mitigations
- **Cost sprawl:** Use small nodes, 1 NAT GW, cleanups cron; stop non‑prod nightly.
- **Secrets leakage:** Use OIDC + fine‑grained IAM; never store secrets in Git; use SSM/Secrets Manager.
- **Drift:** `terraform plan` in CI + `terraform validate` PR gate; periodic `terraform apply -refresh-only` (read‑only in CI).

## 9) Intern Guidelines (Do/Don’t)
**Do:** Write clear commits, PR descriptions, small changes, add README notes, run linters.  
**Don’t:** Push secrets, run `kubectl` on prod without approval, create untagged AWS resources, skip post‑deploy checks.

## 10) Success Metrics
- Lead time from PR merge to prod ≤ 30 minutes (training envs).  
- Rollback time ≤ 5 minutes.  
- CI failure triage ≤ 15 minutes with clear logs.

---

### Appendix A — Reference Repo Structure
```
cloudnautic-aws-devops/
 ├─ app/
 │   ├─ api/ (Flask)
 │   └─ ui/  (React)
 ├─ infra/
 │   ├─ terraform/
 │   │   ├─ envs/{dev,staging,release,prod}/
 │   │   └─ modules/{vpc,eks,ecr,rds,alb,acm,iam,..}
 │   └─ helm/chart/
 ├─ .github/workflows/
 ├─ docs/ (diagrams, runbooks)
 └─ README.md
```
