# Hyperfactory Project Overview

This document provides an overview of the hyperfactory project specification. Each document in this spec folder contains both "decided" and "undecided" sections. The "decided" sections contain specifications that have been finalized, while the "undecided" sections list questions that still need to be answered. Just because a question is in the "undecided" section does not mean that we have not thought about it or have not made a decision--it just means that we have not formally documented the decision yet. And just because a question is in the "decided" section does not mean that it is carved in stone--it just means that it is the current decision.

## Decided

This project has two main parts:

1. **Hyperfactory Core**: A platform inspired by the United Manufacturing Hub that enables factories to collect, process, and analyze data from various sources. It provides a unified namespace for all factory data, visualization tools, and machine learning models. It includes the ability to create custom screens for factory workcenters/machines. It connects to an Odoo database to integrate with an ERP and provide insights into production data. It uses OpenZiti for secure communication between devices and the cloud. On the edge (in the factory), it will use an instance of something like EdgeX Foundry (check the /context folder), which will connect to PLCs, sensors, and other devices, and stream data to a hyperfactory instance running in the cloud on OpenMetal.

2. **Hyperfactory-Cloud (IaaS)**: A cloud-based platform that provides a managed service for hyperfactory. It provides a simple way to deploy and manage hyperfactory instances. It also provides a way to connect hyperfactory instances to each other and to the cloud. Customers will be able to go to an online portal/form/chat (still to be determined), answer some questions about their use case (device IPs/MAC addresses, etc.) and get a customized hyperfactory instance up and running in minutes. They will be able to customize their deployment after creating it from this screen, including paying for more compute, bandwidth, and storage.

## Undecided

- The exact implementation details for each component are documented in the topic-specific files in this directory
- Whether to use Kubernetes or simpler container orchestration (see architecture-infrastructure.md)
- Integration depth and requirements for various components (see individual topic files)
