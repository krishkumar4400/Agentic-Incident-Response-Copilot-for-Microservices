# Agentic Incident Response Copilot for Microservices

## 1. Idea & problem

When a microservices app breaks in production, engineers waste hours manually digging through logs, metrics, and traces across services to find the root cause. This project builds a copilot that watches a running microservices app, detects anomalies, and uses a team of AI agents to investigate the incident the way an SRE would — then proposes (and optionally executes, with approval) a fix.

## 2. Who’d use it

SRE/DevOps/backend teams running anything on Kubernetes. You demo it against your own small sample microservices app, framed as “this is how it’d work for a real production system.”

## 3. Main features

- Live dashboard of logs/metrics/traces for a sample 3–4 service app you deploy

- Anomaly trigger (error rate spike, latency spike) that kicks off an investigation

- Auto-generated incident postmortem after resolution, saved for future reference

- Human-approval gate before any remediation action is actually executed

- Audit trail of every action an agent took

## 4. Where GenAI/Agentic is actually doing work

This is the core of the project, so make it real:

- RAG: your runbooks and past postmortems are embedded in a vector DB. When investigating, agents retrieve relevant past incidents instead of guessing — this is what keeps the system grounded instead of hallucinating a root cause.

- Agentic pipeline, not a single prompt: a Log Analyzer agent summarizes error patterns → a Correlation agent finds related events across services in the time window → a Retrieval agent pulls matching runbook entries → a Root Cause Reasoner chains the evidence into a hypothesis with a confidence score → a Remediation agent proposes an action (e.g. “restart pod X”) but only executes it after a human clicks approve.

- This ReAct-style loop (reason → call a tool → observe → reason again) is what makes it agentic rather than a chatbot answering questions.

## 5. Backend & database

- A “Copilot” service (Spring Boot or Node) exposing REST/WebSocket APIs

- Agent orchestration logic (LangGraph or similar) as its own service

- Postgres for incident metadata, pgvector (or Pinecone) for runbook embeddings, Redis for agent-run state, Prometheus for metrics

- A simple job queue (SQS or BullMQ) to decouple anomaly detection from the agent pipeline

## 6. DevOps & cloud

- Everything in Docker, deployed to EKS via Helm
Terraform for VPC, EKS, RDS, ElastiCache, IAM roles

- GitHub Actions: lint/test → build & push to ECR → Terraform plan/apply → Helm deploy

- Prometheus + Grafana for observability (this also doubles as the data the agent reads)

- Security: scoped Kubernetes RBAC so the agent’s service account can only restart/scale specific deployments — never delete anything; secrets in AWS Secrets Manager

- For live demos, you can run the cluster on a small/spot setup or k3s to avoid cost, and mention that trade-off explicitly in interviews

## 7. Why it’s impressive

Most student AI projects are a chatbot over some docs. This shows you can reason about production reliability, safe autonomous action-taking, and real infra — the combination of agentic AI + platform engineering is rare and directly relevant to DevOps/platform/backend roles.

## 8. What you can discuss in interviews

Multi-agent design and failure modes, how you prevent hallucinated root causes (grounding via RAG), why you put a human approval gate before destructive actions, least-privilege RBAC for agent tool use, how you’d scale log ingestion with Kafka, LLM cost control at scale, and how you evaluated agent accuracy against a labeled set of past incidents.
