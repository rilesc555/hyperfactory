# Hyperfactory

This document keeps track of the spec for the hyperfactory project. The first part of the document, "decided", is the spec that has been decided on. The second part of the document, "undecided", is the spec that has not been decided on yet--this is a list of questions that we still have to answer. Just because a question is in the "undecided" section does not mean that we have not thought about it or have not made a decision--it just means that we have not formally documented the decision yet. And just because a question is in the "decided" section does not mean that it is carved in stone--it just means that it is the current decision.

## Decided

This project has two main parts:

1. A core product called hyperfactory. This a platform inspired by the United Manufacturing Hub that enables factories to collect, process, and analyze data from various sources. It provides a unified namespace for all factory data, visualization tools, and machine learning models. It includes that ability to create custom screens/ for factory workcenters/machines. It connects to an odoo database to integrate with an ERP and provide insights into production data. It uses openziti for secure communication between devices and the cloud. On the edge (in the factory), it will use an instance of something like edgex-foundry (check the /context folder), which will connect to PLCs, sensors, and other devices, and stream data to a hyperfactory instance running in the cloud on OpenMetal

2. An IaaS called hyperfactory-cloud. This is a cloud-based platform that provides a managed service for hyperfactory. It provides a simple way to deploy and manage hyperfactory instances. It also provides a way to connect hyperfactory instances to each other and to the cloud. Customers will be able to go to an online portal/form/chat (still to be determined), answer some questions about their use case (device ips/mac addresses, etc.) and get a customized hyperfactory instance up and running in minutes. They will be able to customize their deployment after creating it from this screen, including paying for more compute, bandwidth, and storage.

In the /context folder, there is a folder called infra that is being used in another project to control the kubernetes cluster for another project. We might use a k8s cluster, or we might not

## Undecided

### Architecture & Infrastructure
1. Do we use Kubernetes for hyperfactory core, or a simpler container orchestration approach (Docker Compose, Nomad, etc.)?
2. What is the exact role of EdgeX Foundry vs. Benthos-UMH for edge data collection? Do we fork/extend one of these?
3. How do we handle multi-tenancy in hyperfactory-cloud? Separate clusters per customer, namespaces, or database-level isolation?
4. What message broker do we use? (Redpanda, Kafka, NATS, MQTT broker?)
5. Do we need a time-series database? (TimescaleDB, InfluxDB, QuestDB?) Or can we use standard PostgreSQL?
6. What's our data retention strategy? Hot/warm/cold storage tiers?
7. How do we handle edge-to-cloud synchronization when connectivity is intermittent?

### OpenZiti Integration
1. How deeply integrated is OpenZiti? Is it required or optional?
2. Do edge devices connect directly through OpenZiti tunnels, or do we have a gateway pattern?
3. How do we handle OpenZiti identity provisioning during customer onboarding?
4. What's the fallback if OpenZiti has issues?

### Odoo Integration
1. Which Odoo modules do we integrate with? (Manufacturing, Inventory, MRP, Quality?)
2. Is Odoo integration required or optional?
3. Do we use Odoo's API, or do we need direct database access?
4. How do we handle Odoo version compatibility (Community vs. Enterprise, different versions)?
5. Do we push data to Odoo, pull from Odoo, or both?

### Edge Deployment
1. What hardware requirements for edge devices? (CPU, RAM, storage)
2. What OS do we support? (Linux only, Windows, both?)
3. How do we handle edge device provisioning and initial setup?
4. Do we support air-gapped or offline-first edge deployments?
5. How do we handle edge device updates and configuration management?
6. What happens when edge devices lose cloud connectivity?

### Data Collection & Processing
1. What industrial protocols must we support? (OPC UA, Modbus TCP/RTU, EtherNet/IP, PROFINET, S7, MTConnect, etc.)
2. How do we handle protocol discovery and auto-configuration?
3. What's our data model for the unified namespace? (ISA-95, custom, UMH-style?)
4. How do we handle data transformation and normalization at the edge vs. cloud?
5. What's our approach to real-time vs. batch processing?
6. How do we handle data quality, validation, and anomaly detection?

### Visualization & UI
1. What technology stack for custom screens? (React, Vue, Svelte? Low-code platform?)
2. Do we build a custom dashboard builder, or integrate with Grafana/similar?
3. How do users create and customize screens for workcenters/machines?
4. Do we need mobile apps, or is web-only sufficient?
5. What's the user permission model? (Role-based, attribute-based?)

### Machine Learning & Analytics
1. What ML capabilities do we provide out of the box? (Predictive maintenance, anomaly detection, quality prediction?)
2. Do we train models in the cloud, at the edge, or both?
3. What ML framework/platform? (TensorFlow, PyTorch, MLflow, custom?)
4. How do users deploy custom models?
5. Do we need real-time inference at the edge?

### Hyperfactory-Cloud (IaaS)
1. What's the onboarding flow? (Portal, form, chat, wizard?)
2. How do we handle billing? (Usage-based, tiered plans, custom quotes?)
3. What payment processing? (Stripe, custom?)
4. How do customers manage their instances? (Web console, CLI, API, all three?)
5. Do we offer a free tier or trial period?
6. What SLAs do we commit to?
7. How do we handle customer data backup and disaster recovery?
8. Do we support customer-managed encryption keys?

### OpenMetal Deployment
1. What OpenMetal services do we use? (Bare metal, private cloud, managed Kubernetes?)
2. How do we handle scaling across multiple OpenMetal regions?
3. What's our deployment automation? (Terraform, Ansible, Pulumi, custom?)
4. How do we handle infrastructure monitoring and alerting?

### Development & Operations
1. What programming languages for each component? (Go, Python, Rust, TypeScript?)
2. What's our testing strategy? (Unit, integration, e2e?)
3. How do we handle versioning and upgrades?
4. What's our CI/CD pipeline?
5. How do we handle secrets management? (Vault, cloud provider, custom?)
6. What observability stack? (Prometheus, Grafana, Loki, Tempo? OpenTelemetry?)

### Security & Compliance
1. What security certifications do we need? (SOC 2, ISO 27001, etc.)
2. How do we handle customer data privacy? (GDPR, CCPA compliance?)
3. What's our vulnerability management process?
4. Do we support SSO/SAML for enterprise customers?
5. How do we handle audit logging?

### Business & Go-to-Market
1. What's our pricing model?
2. Who is our target customer? (SMB manufacturers, enterprises, specific industries?)
3. What's our support model? (Community, email, phone, dedicated support engineer?)
4. Do we offer professional services for custom integrations?
5. What's our documentation strategy?
6. Do we need a partner/reseller program?

### Open Source vs. Proprietary
1. Is hyperfactory core open source, proprietary, or open-core?
2. If open-core, what features are open vs. paid?
3. What license do we use?
4. How do we handle community contributions?