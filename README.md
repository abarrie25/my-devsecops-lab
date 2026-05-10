# Alpha Yerroh Barrie — Cloud, DevOps & DevSecOps Portfolio

**Location:** Placentia, California, USA
**Focus:** AWS, DevSecOps, Kubernetes, SRE, Platform Engineering
**Certifications:** AWS Certified Solutions Architect – Associate
**Target Roles:** DevOps Engineer • DevSecOps Engineer • AWS Cloud Engineer • SRE • Platform Engineer • Infrastructure Engineer • Cloud Security Engineer • CI/CD Engineer • Linux Systems Engineer • Production/Cloud Support Engineer

---

## How To Use This Portfolio

Each project below is written as if I built it inside a real company, with real operational pain, real fixes, and measurable outcomes. Every project includes:

1. Simple Human Explanation
2. Real-World Company Scenario
3. Architecture Overview
4. AWS & DevOps Tools Used
5. Security Design
6. CI/CD Pipeline
7. Monitoring & Observability
8. Incident Scenario & Response
9. Step-by-Step Build Plan
10. GitHub Repository Structure
11. README Structure
12. Resume Bullet Points
13. LinkedIn Portfolio Description
14. Recruiter Appeal
15. FAANG Interview Questions
16. Common Mistakes
17. Lessons Learned

The flagship project (Project 01) is the deepest. The remaining 30 projects are written as believable, role-targeted portfolio pieces I can actually build, document, and defend in interviews.

**Role coverage map (minimum 3 projects per target role):**

| Role | Projects |
|---|---|
| DevOps Engineer | 01, 02, 05, 08, 17, 22 |
| DevSecOps Engineer | 01, 03, 06, 10, 13, 19, 24 |
| AWS Cloud Engineer | 01, 02, 04, 09, 16, 25 |
| Cloud Engineer | 01, 04, 09, 16, 20 |
| Site Reliability Engineer (SRE) | 01, 07, 11, 18, 23, 28 |
| Platform Engineer | 01, 05, 08, 14, 21, 30 |
| Infrastructure Engineer | 01, 02, 04, 15, 20, 25 |
| Cloud Security Engineer | 01, 03, 06, 12, 19, 24, 27 |
| CI/CD Engineer | 02, 05, 10, 13, 17, 22 |
| Linux Systems Engineer | 15, 20, 26, 29 |
| Production Support Engineer | 07, 11, 18, 23, 26 |
| Cloud Support Engineer | 04, 16, 26, 29, 30 |

---

# 🏛️ FLAGSHIP PROJECT

## 01. `aws-secure-campus-platform` — Secure Cloud-Native University Management System

### 1. Simple Human Explanation
A full university management platform on AWS EKS that handles student enrollment, grades, fee/payment processing, staff access, and audit logging — built the way a real SaaS EdTech company would build it. It enforces RBAC, detects unauthorized access, ships logs to a SIEM, scans every container for vulnerabilities, and fails over to a second region during a disaster. EdTech platforms get hit hard (credential stuffing, grade tampering, fee fraud), so I wanted a realistic environment to practice end-to-end DevSecOps ownership.

### 2. Real-World Company Scenario
A mid-sized California university (30k students, 8k staff) is migrating from an on-prem Oracle/IIS stack to AWS after a ransomware incident exposed a flat, over-privileged network. Leadership wants strict RBAC, fee-processing PCI-aware controls, FERPA-aligned audit logging, automated backups, and a DR plan auditors will accept. I act as the DevSecOps engineer who owns build, deploy, security, and on-call.

### 3. Architecture Overview
- **Frontend:** React SPA on CloudFront + S3, WAF in front, ACM-signed cert.
- **API:** Node.js + Python microservices (auth, students, grades, billing, notifications) on EKS across 3 AZs.
- **Data:** RDS PostgreSQL Multi-AZ (students, grades), DynamoDB (sessions, audit events), ElastiCache Redis (hot reads), S3 for transcripts/receipts (SSE-KMS, Object Lock for audit immutability).
- **Identity:** Cognito for student/staff auth, IAM Identity Center for engineer access, IRSA for pod-level AWS permissions.
- **Edge/Security:** Route 53 health-checked failover to us-west-2, CloudFront + AWS WAF (managed + custom rules), Shield Standard.
- **Observability:** Prometheus + Grafana in-cluster, CloudWatch for AWS primitives, OpenSearch for logs, Falco for runtime, GuardDuty + Security Hub + Config for posture.
- **DR:** Active-passive across us-east-1 (primary) and us-west-2 (warm standby), RDS cross-region read replica, S3 CRR, Route 53 failover, Terraform-driven promotion runbook.

### 4. AWS & DevOps Tools Used
AWS: EKS, EC2, VPC, Transit Gateway, RDS, DynamoDB, S3, CloudFront, Route 53, ACM, WAF, Shield, Cognito, IAM Identity Center, KMS, Secrets Manager, Systems Manager, GuardDuty, Security Hub, Config, CloudTrail, CloudWatch, SNS, SQS, EventBridge, Backup.
DevOps: Terraform (modular), Helm, ArgoCD, GitHub Actions, Jenkins (heavy build matrix), Docker, Trivy, SonarQube, OWASP ZAP, Checkov, tfsec, Vault (on EKS, dynamic DB creds), Falco, Kyverno, OPA Gatekeeper, Prometheus, Grafana, Loki, Fluent Bit, Velero, Litmus (chaos), Python + Bash automation.

### 5. Security Design
- **Identity:** Cognito with MFA for students/staff; IAM Identity Center permission sets for engineers; no long-lived IAM users; every service uses IRSA with policies scoped to specific resource ARNs.
- **Network:** Private EKS API endpoint, nodes in private subnets, VPC endpoints for S3/ECR/Secrets Manager/STS, service-to-service security groups, no 0.0.0.0/0 ingress except ALB + Cognito Hosted UI.
- **Secrets:** Vault issues short-lived DB creds (15 min TTL) via the Vault Agent Injector; GitHub Actions uses OIDC-federated AWS roles (zero static keys); per-env KMS CMKs with rotation.
- **Runtime:** Kyverno blocks privileged pods, hostPath mounts, `latest` tags, and pods without resource limits; Falco alerts on shell-in-container, unexpected egress, and mining indicators; GuardDuty for VPC/EKS threats.
- **Data:** RDS KMS-encrypted, S3 SSE-KMS + Object Lock (compliance) on audit bucket, TLS 1.2+ end-to-end, DynamoDB encryption at rest, PII column-encrypted with envelope keys.
- **Audit:** Org CloudTrail → S3 (Object Lock) → OpenSearch; app audit events (who viewed which student record) via Fluent Bit to OpenSearch with 400-day retention.
- **Compliance:** AWS Config rules mapped to CIS AWS + FERPA-aligned controls; Security Hub central view; nightly Prowler + ScoutSuite scans.

### 6. CI/CD Pipeline
Repo-per-service, monorepo for infra. GitHub Actions with OIDC to AWS:
1. Lint & unit test (pytest / jest / golangci-lint).
2. SAST: SonarQube quality gate (fails on new critical/major).
3. SCA: Trivy fs + Trivy image, Grype on SBOM (Syft).
4. IaC scan: Checkov + tfsec on Terraform and Helm.
5. Build: multi-stage Docker, distroless base, Cosign-signed image, SBOM attached.
6. Push: ECR with immutable `<git-sha>` tags, no `latest`.
7. DAST (staging): OWASP ZAP baseline against preview env.
8. Deploy: ArgoCD picks up image tag bump in env repo → Argo Rollouts canary 10% → 50% → 100% gated by Prometheus success-rate and p95 latency.
9. Post-deploy: smoke tests, synthetic checks, Grafana annotation.

Jenkins runs heavy integration tests on ephemeral EKS namespaces (spot) where matrix parallelism matters.

### 7. Monitoring & Observability
- **Metrics:** Prometheus scrapes services, node-exporter, kube-state-metrics, CloudWatch agent for RDS/ALB. Grafana dashboards per service with RED + USE panels.
- **Logs:** Fluent Bit → Loki for app logs, CloudWatch → OpenSearch for AWS logs, org CloudTrail for audit.
- **Traces:** OpenTelemetry SDK → AWS X-Ray + Tempo.
- **SLOs:** 99.9% on `/login`, 99.95% on `/grades/read`, p95 < 400ms on `/payments/checkout`. Error budgets in Grafana with burn-rate alerts (fast: 2% in 1h, slow: 10% in 6h).
- **Alerting:** Alertmanager → PagerDuty (Sev1/Sev2), Slack (Sev3). Every alert links to a `/docs/runbooks` runbook.

### 8. Incident Scenario & Response
**Simulated incident:** Midterm week, `/grades/read` p95 spikes to 3.8s, 5xx climbs to 4%. PagerDuty pages at 02:14.
- **Triage (5 min):** Grafana shows RDS CPU 96%, connections 480/500. CloudWatch Logs Insights shows a new query joining `grades` to `enrollments` without an index.
- **Mitigate (10 min):** Feature-flag the new transcript export endpoint off; connections drop to 210, p95 back to 350ms.
- **Fix next morning:** Add composite index, re-enable flag, canary at 5%, watch error budget.
- **Postmortem:** Blameless `/docs/postmortems/2026-03-14-grades-latency.md`. Actions: require `EXPLAIN` on new DB queries in PR template, add `pg_stat_statements` dashboard, preflight load test for new joins against `students`.

### 9. Step-by-Step Build Plan
1. Bootstrap AWS Organization, dev/stage/prod accounts, Control Tower, SCPs.
2. Terraform remote state (S3 + DynamoDB lock), separate backend per account.
3. Build `modules/vpc`, `modules/eks`, `modules/rds`, `modules/ecr`, `modules/irsa`.
4. Stand up EKS in dev, install ArgoCD, Kyverno, Vault, cert-manager, external-dns.
5. Containerize 5 services, push to ECR, write Helm charts.
6. Wire GitHub Actions with OIDC; no static AWS keys.
7. Build env repo (ArgoCD ApplicationSets per service per env).
8. Add Prometheus/Grafana/Loki via kube-prometheus-stack.
9. Onboard Trivy, SonarQube, Checkov, Cosign, Falco, GuardDuty, Security Hub.
10. Wire Cognito + WAF + CloudFront + Route 53.
11. Build DR runbook, tabletop, then live failover test.
12. Chaos game day with Litmus (kill node, kill RDS primary, block an AZ).
13. Document everything, write postmortems for simulated incidents.

### 10. GitHub Repository Structure
```
aws-secure-campus-platform/
├── README.md
├── docs/
│   ├── architecture/
│   ├── runbooks/
│   ├── postmortems/
│   ├── threat-model.md
│   └── dr-plan.md
├── infra/
│   ├── terraform/
│   │   ├── modules/{vpc,eks,rds,irsa,waf,route53,backup}
│   │   └── live/{dev,stage,prod}/{us-east-1,us-west-2}
│   └── policies/
├── platform/
│   ├── argocd/
│   ├── helm/{base,overlays}
│   ├── kyverno/
│   ├── vault/
│   └── observability/
├── services/
│   ├── auth-service/
│   ├── student-service/
│   ├── grades-service/
│   ├── billing-service/
│   └── notify-service/
├── .github/workflows/
├── scripts/
└── tests/{chaos,load,dast}
```

### 11. README Structure
1. One-paragraph pitch + architecture diagram
2. Screenshots (Grafana, ArgoCD, WAF, Security Hub)
3. Stack table (AWS / DevOps / Security)
4. Threat model summary
5. SLOs and error budgets
6. Deploy instructions
7. Security controls matrix (mapped to CIS / FERPA-aligned)
8. DR plan + tested RTO/RPO
9. Incident library (postmortem links)
10. Monthly cost estimate
11. Lessons learned

### 12. Resume Bullet Points
- Designed and built a multi-region, EKS-based student management platform on AWS supporting 30k simulated users with 99.95% availability SLO and tested 18-minute RTO cross-region failover.
- Implemented a zero-trust DevSecOps pipeline (Trivy, SonarQube, Checkov, OWASP ZAP, Cosign, Kyverno) reducing critical vulnerabilities shipped to prod to zero across 5 services.
- Built Terraform modules and ArgoCD app-of-apps that cut environment provisioning from ~2 days of manual work to 35 minutes end-to-end.
- Authored 12 runbooks and ran a chaos game day with Litmus; reduced simulated MTTR from 48 min to 11 min through dashboard and runbook improvements.
- Rolled out IRSA, Vault dynamic DB credentials, and Cognito MFA, eliminating all long-lived service credentials from the platform.

### 13. LinkedIn Portfolio Description
Built a production-grade, cloud-native university management platform on AWS EKS with full DevSecOps ownership: Terraform IaC, ArgoCD GitOps, Cognito + IAM Identity Center, Kyverno + Falco runtime security, Prometheus + Grafana observability, and a tested cross-region DR plan. Wrote the runbooks, ran chaos experiments, and documented every incident like a real on-call engineer.

### 14. Recruiter Appeal
Hits every keyword recruiters search for (EKS, Terraform, ArgoCD, Kyverno, Vault, Prometheus, GuardDuty, DR, SLOs) and tells a story: the candidate thought about attackers, on-call engineers, auditors, and cost. That is a senior mindset in a mid-level candidate — the exact profile that gets shortlisted.

### 15. FAANG Interview Questions
- Walk me through a request from a student's browser to the grades DB and what protects it at each hop.
- How do you rotate database credentials without downtime?
- Your Prometheus alerted on burn-rate. What is burn-rate and why use it over static thresholds?
- us-east-1 is fully down. Walk me through failover and what is likely to break.
- How do you keep devs fast while enforcing Kyverno and scanning gates?
- A pod is egressing to a suspicious IP. Walk me through detection → containment → eradication.

### 16. Common Mistakes
- Public EKS API "secured by IAM" — still needs to be private.
- Helm `latest` tags making rollbacks unpredictable.
- Treating CloudTrail as alerting; it is forensic.
- Skipping Kyverno because it feels noisy; it is the cheapest guardrail.
- Forgetting cross-region IAM and KMS key policies during DR.

### 17. Lessons Learned
Security is cheapest when it is a pipeline stage, not a meeting. SLOs force honest scope conversations. Runbooks written at 3 a.m. are worse than runbooks written Tuesday at 2 p.m. Canary + fast rollback beats perfect pre-prod testing. Every AWS service has a quota that will bite at the worst time — document them.

---
