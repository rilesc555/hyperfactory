# Hyperfactory Specification Documentation

This directory contains the complete specification documentation for the hyperfactory project, organized by topic area. Each document follows a consistent structure with **Decided** and **Undecided** sections to track the current state of decisions.

## Document Structure

Each specification document contains:

- **Decided**: Finalized specifications and architectural decisions
- **Undecided**: Open questions and decisions that still need to be made

> **Note**: Items in the "Decided" section represent current decisions but are not carved in stone. Items in the "Undecided" section may have been discussed but haven't been formally documented yet.

## Project Overview

Hyperfactory consists of two main components:

1. **Hyperfactory Core**: A manufacturing data platform inspired by the United Manufacturing Hub
2. **Hyperfactory-Cloud**: An IaaS platform providing managed hyperfactory instances

## Specification Documents

### Core Architecture
- **[overview.md](./overview.md)** - Project overview and high-level architecture decisions
- **[architecture-infrastructure.md](./architecture-infrastructure.md)** - Container orchestration, databases, and infrastructure decisions

### Integration & Connectivity
- **[openziti-integration.md](./openziti-integration.md)** - Secure communication and networking
- **[odoo-integration.md](./odoo-integration.md)** - ERP system integration
- **[data-collection-processing.md](./data-collection-processing.md)** - Industrial protocols and data processing

### Platform Components
- **[edge-deployment.md](./edge-deployment.md)** - Edge device requirements and deployment
- **[visualization-ui.md](./visualization-ui.md)** - User interfaces and visualization tools
- **[machine-learning-analytics.md](./machine-learning-analytics.md)** - ML capabilities and analytics

### Cloud & Operations
- **[hyperfactory-cloud-iaas.md](./hyperfactory-cloud-iaas.md)** - Managed service platform
- **[openmetal-deployment.md](./openmetal-deployment.md)** - Cloud infrastructure deployment
- **[development-operations.md](./development-operations.md)** - Development practices and operations

### Business & Compliance
- **[security-compliance.md](./security-compliance.md)** - Security requirements and compliance
- **[business-go-to-market.md](./business-go-to-market.md)** - Business model and market strategy
- **[open-source-vs-proprietary.md](./open-source-vs-proprietary.md)** - Licensing and open source strategy

## How to Use This Documentation

1. **Start with [overview.md](./overview.md)** for the big picture
2. **Review topic-specific documents** based on your area of interest
3. **Check "Decided" sections** for current architectural decisions
4. **Review "Undecided" sections** for areas that need attention
5. **Update documents** as decisions are made by moving items from "Undecided" to "Decided"

## Contributing to the Specification

When updating these documents:

1. Move finalized decisions from "Undecided" to "Decided" sections
2. Add new questions to "Undecided" sections as they arise
3. Update "Decided" sections when architectural decisions change
4. Maintain consistency across related documents
5. Keep the overview document synchronized with major changes
