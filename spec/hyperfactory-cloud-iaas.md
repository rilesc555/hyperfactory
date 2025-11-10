# Hyperfactory-Cloud (IaaS)

## Decided

### 1. Product Definition
- Managed IaaS for per-customer Hyperfactory instances on OpenMetal
- 10-minute provisioning from signup to first data
- Simplicity first: Docker Compose (not Kubernetes)

### 2. Tenancy & Isolation
- Per-customer OpenMetal projects; strong network and billing isolation
- Each project runs a standard Docker Compose stack
- Control plane runs in a dedicated OpenMetal project

### 3. Control Plane & Lifecycle
- Control API: authN/Z, customer accounts, quotas, rate limiting
- Infrastructure Orchestrator: OpenMetal API for provision/resize/suspend/teardown
- Billing connector: Odoo integration for real-time invoicing and payments
- Global monitoring: fleet health, usage metering, alerts

### 4. Instance Topology (per customer)
- UMH Core (cloud) + Redpanda (Kafka-compatible bus)
- PostgreSQL + TimescaleDB
- Grafana + customer dashboard
- Odoo Agent
- OpenZiti components as needed for secure remote access
- Hyperfactory Agent (telemetry and remote management)

### 5. Onboarding & Provisioning
- Self-service web portal wizard for MVP
- Optional sandbox/trial with resource caps
- Payment confirmation via Odoo triggers automated provisioning

### 6. Scaling & Quotas
- Self-service vertical scaling (vCPU/RAM/storage) and node size changes
- Quotas and hard limits per customer; preflight capacity checks
- Real-time cost preview and post-change billing via Odoo

### 7. Data Movement & Connectivity
- One-way edge → cloud via Redpanda Connect core-to-core bridge
- MQTT remains device-facing at the edge only
- mTLS between edge and cloud; per-tenant CA and ACLs
- No cloud → edge replication by default

### 8. Security
- Per-customer isolation at infra, network, and data layers
- DB row-level security and encryption at rest; secrets managed by control plane
- Access via SSO-ready customer portal; audit logging of admin actions

### 9. Observability & Support
- Central metrics/logs, per-tenant dashboards, alerting
- Status page and incident comms playbook
- Support tiers: Standard and Premium (response targets defined)

### 10. Backup & DR
- Daily PostgreSQL backups + WAL for PITR; Ceph 3x replication
- Optional cross-region backup add-on
- Recoverability validated via periodic restore tests

### 11. SLO/SLA Targets (initial)
- Control plane availability: 99.9% monthly
- Customer instance SLO: 99.5% (non-HA baseline)
- RPO ≤ 15 minutes; RTO ≤ 4 hours (non-HA), better with premium add-ons

Cross-references: see [architecture-infrastructure.md](./architecture-infrastructure.md), [openmetal-deployment.md](./openmetal-deployment.md), [security-compliance.md](./security-compliance.md), and [openziti-integration.md](./openziti-integration.md).

## Undecided

1. Onboarding UX details (portal wizard vs. conversational chat) and AI-assist scope
2. Pricing model and packaging (tiers vs. usage), committed discounts, annual prepay
3. Payment processor choice (Odoo-native vs. Stripe) and reconciliation flow
4. Region selection and data residency controls per customer
5. High-availability options and premium tiers; dedicated hardware projects
6. Secrets store selection and CMEK/KMS support for customer-managed keys
7. Observability stack specifics (Prometheus/Grafana/Loki/OpenTelemetry)
8. Support SLAs and escalation playbook; pager coverage hours
9. Incident communication and public status page policy
10. Deprovisioning flow, data export, and retention periods
11. Customer-facing API/CLI surface and RBAC model; SSO providers supported
12. Default quotas and rate limits; burst policy
13. Egress pricing and bandwidth shaping policies
14. Compliance roadmap (SOC 2/ISO 27001) and required controls
15. Key/cert rotation cadence and CA operations

