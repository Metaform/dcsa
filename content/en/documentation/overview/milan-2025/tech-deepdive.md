---
title: "Technical Deep Dive EDC"
linkTitle: "DeepDive"
weight: 1100
description: >-
     Technical Deep Dive EDC - June Milan 2025 event
---
📅 June 30, 2025 | 🕖 07:41  
**Presenter:** Jim Marino

## Jim’s Component Walkthrough Summary

### 🔄 Two-Tier Architecture: Control Plane vs Industrial Data Plane
- **Control Plane:** Lightweight, policy-driven layer managing catalog, contract negotiation, and enforcement.
- **Data Plane:** Handles big-data, file transfers, and streaming workloads.
- Can be split across clouds or run side-by-side, with **DSP messages keeping both planes in sync**.

---

### 🧱 Industrial Data-Plane Blueprint
- Reference stack combines:
  - **On-prem ingest pipeline** (OPC UA, AAS, edge filtering, blob/SQL storage)
  - **Multi-tenant SaaS layer** exposing assets over REST or AAS APIs
- Designed for **connector-based transfer**

---

### 🔐 DSP + DCP Protocols
- **Dataspace Protocol (DSP):** Orchestrates `catalog → contract → transfer`
- **Decentralized Claims Protocol (DCP):** Adds verifiable credentials so **only trusted partners** get access — no central identity provider required

---

### 🧪 Virtualised EDC Runtime (EDC-V)
- A **virtualisation container** enables:
  - Many tenants + many dataspaces
  - One EDC instance per environment
- Context loaded **per request**, with message queues ensuring **scalability and elasticity**

---

### 🚀 Efficiency Boost via State Machines
- Swaps inefficient polling ("tick-over") for **reliable message queues**
- Enables horizontally-scalable connector functions and reduces CPU waste
- **JVM overhead amortised across tenants**

---

### 🧵 Connector Fabric Manager (CFM)
- **Tenant & Ops Manager**:
  - Tracks tenants, memberships, DIDs
  - Declarative “deployment definitions” for provisioning clusters
- Pluggable agents deploy or migrate clusters (e.g. Kubernetes cells)

---

### ⚙️ Built-in Workload Automation
- Tasks like startup, scale, failover, migration simplified to:
  - **Copy config + route traffic**
  - Infrastructure handled outside EDC using **CFM provider plugins**

---

### 🛡 Policy Engine Upgrade
- Dynamic **CEL-based evaluator** supports **runtime rules per dataspace**
  - e.g., apply different access rules for Catena-X vs. Manufacturing-X
- Enables multi-tenant, policy-isolated runtimes

---

### 🏛 Standards & Governance Alignment
- Built to align with:
  - **ISO 20151**, **CEN JTC 25 Trusted Data Transactions**
  - **Eclipse Dataspace Working Group** specs
- Supports procurement-ready governance and future-proof interoperability

---

### 🧪 Next Steps for Partners
- Help **dogfood the virtualised build**
- Contribute **data-plane plugins**
- Integrate **CFM with your cloud**
- Public **Minimum Viable Dataspace repo and demo clusters are available**

🔗 **Demo Repository:**  
[github.com/paullatzelsperger/MinimumViableDataspace/tree/feat/aas_demo](https://github.com/paullatzelsperger/MinimumViableDataspace/tree/feat/aas_demo)
