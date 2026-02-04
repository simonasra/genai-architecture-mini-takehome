# Company Context

## About Metrica

Metrica is a B2B SaaS analytics platform that helps product teams understand user behavior. The platform tracks events, computes metrics, and provides dashboards for decision-making.

Metrica has ~200 customers ranging from startups to mid-market companies. Each customer has their own isolated data environment.

---

## The Analytics-Aware Chatbot Project

The product team wants to build an internal chatbot that allows employees to ask natural language questions about product analytics. Instead of writing SQL or navigating dashboards, users can simply ask:

> "What was our DAU last week?"

The chatbot should translate questions into data queries, fetch results, and respond conversationally.

---

## User Roles

Three internal roles will use the chatbot:

| Role | Description | Intended Data Access |
|------|-------------|---------------------|
| SupportAgent | Handles customer tickets | Aggregate metrics only (no customer-specific data) |
| Analyst | Builds reports and investigates trends | Full analytics access, but no PII |
| Executive | Reviews high-level performance | Dashboard summaries and KPIs only |

All users authenticate via SSO. Role information is available in the session token.

---

## Data Sources

The chatbot can potentially access three data sources:

### 1. Event Warehouse

A columnar database containing raw event data:

- `events` table: `timestamp`, `event_name`, `user_id`, `session_id`, `properties` (JSON)
- `sessions` table: `session_id`, `user_id`, `start_time`, `end_time`, `device`, `country`
- Billions of rows, updated in near real-time

### 2. Metrics API

Pre-computed KPIs served via REST API:

- Daily/weekly/monthly aggregates: DAU, WAU, MAU, retention, conversion rates
- Cached and fast (p95 < 100ms)
- No customer-level breakdowns

### 3. Customer Table

Relational database containing customer account information:

| Column | Example | Sensitivity |
|--------|---------|-------------|
| `customer_id` | `cust_12345` | Internal ID |
| `company_name` | `Acme Corp` | Business data |
| `contact_email` | `jane@acme.com` | PII |
| `plan_tier` | `enterprise` | Business data |
| `mrr` | `4500.00` | Confidential |
| `created_at` | `2024-03-15` | Metadata |

---

## Constraints

The system must operate within these boundaries:

| Constraint | Value | Notes |
|------------|-------|-------|
| p95 Latency | 3 seconds | Users expect near-instant responses |
| Cost per conversation | $0.50 max | Budget sustainability at scale |
| Log retention | 30 days | Compliance and storage costs |
| Availability | 99.5% uptime | Internal tool, not customer-facing |
| PII handling | Must not expose to unauthorized roles | Required by customer DPAs |
| Audit logs | Required for data access | GDPR compliance |
