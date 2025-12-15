---
title: "Eclipse Data Components Status"
linkTitle: "Project Status"
weight: 1
description: >-
  Status of the EDC release for the Data Spaces Symposium Event in Madrid on 10/11th of February.
---
# Eclipse Dataspace Components – Madrid Development Dashboard for February

## Purpose
- Status view for the February 2026 Madrid demo, focused on trusted data sharing based on Eclipse Dataspace Components (EDC), DSP and DCP.
- Audience: Extended Project Dali Team.
- Scope: Component list with GitHub links, current status, and what is realistically in scope for the February release.

## Status Legend
- **GREEN** – On track for the scope for the February release.
- **ORANGE** – Some risks.
- **RED** – Not able to make the deadlines.

---

## 1. Component List with Tracking Links

| # | Component | Main Repository | Issues / Board | Status | Notes |
|---|-----------|----------------|----------------|--------|-------|
| 1 | Connector Fabric Manager (CFM) | https://github.com/Metaform/connector-fabric-manager | https://github.com/Metaform/connector-fabric-manager/issues | GREEN | Tenant + workflow backbone for February |
| 2 | EDC Virtual Connector (EDC-V) | https://github.com/eclipse-edc/Virtual-Connector | https://github.com/eclipse-edc/Virtual-Connector/issues | GREEN | Virtualised control plane, classic data plane used for February |
| 3 | Identity Hub (Wallet) | https://github.com/eclipse-edc/IdentityHub | https://github.com/eclipse-edc/IdentityHub/issues | GREEN | Verifiable credentials, certificates use case |
| 4 | Classic EDC Connector | https://github.com/eclipse-edc/Connector | https://github.com/eclipse-edc/Connector/issues | GREEN | Control + data plane, fallback runtime for February |
| 5 | Data Plane Signaling (DPS) | https://github.com/eclipse-dataplane-signaling/dataplane-signaling | https://github.com/eclipse-dataplane-signaling/dataplane-signaling/issues | GREEN | Spec and reference implementation progressing |
| 6 | Data Plane Core / SDKs | https://github.com/eclipse-dataplane-core | Issues per SDK repo | GREEN | Available, used experimentally alongside classic data plane |
| 7 | Protocol TCKs (DSP / DCP / DPS) | https://github.com/eclipse-dataspacetck/dsp-tck | Issues per TCK repo | GREEN | Protocol conformance validation |
| 8 | Participant & CSP Portals (UI) | To be contributed (Aruba UI + CFM frontend) | n/a | ORANGE | Essential onboarding and basic catalog/transfer flows |
| 9 | Industrial Data Plane & OPC UA Cloud Library | https://github.com/OPCFoundation/UA-CloudLibrary | https://github.com/OPCFoundation/UA-CloudLibrary/milestone/2 | ORANGE | Scenario-specific integration for February |

---

## 2. Per-Component Notes

### 1. Connector Fabric Manager (CFM)
- **Role:** Tenant and lifecycle manager, enabling connector-as-a-service for multiple organisations.
- **February:** Used to onboard demo tenants, including DNS setup, EDC-V provisioning, wallet creation, and sample credentials.

### 2. EDC Virtual Connector (EDC-V)
- **Role:** Virtualised control plane for trusted data sharing, externalising the data plane via DPS.
- **February:** Demonstrates multi-tenant control plane behaviour, with actual transfers still using the classic data plane to reduce risk.

### 3. Identity Hub (Wallet Service)
- **Role:** Management of decentralised identifiers (DIDs) and verifiable credentials (e.g. ISO certificates, membership attestations).
- **February:** Central to the certificates-for-trust scenario.

### 4. Classic EDC Connector
- **Role:** Proven combined control and data plane, used in existing Catena-X deployments.
- **February:** Responsible for actual data transfers behind the scenes, even when orchestrated by the new architecture.

### 5. Data Plane Signaling (DPS)
- **Role:** Standard protocol between control planes (e.g. EDC-V) and independent data plane services.
- **February:** Specification and prototype available, but not the primary transfer path.

### 6. Data Plane Core / SDKs
- **Role:** SDKs and building blocks for implementing DPS-compliant data planes.
- **February:** Used for experiments and early demos, not yet the sole transfer mechanism.

### 7. Protocol TCKs (DSP / DCP / DPS)
- **Role:** Technology Compatibility Kits validating protocol conformance.
- **February:** Used for internal validation, not directly visible in the demo UI.

### 8. Portals / UI (Participant and CSP Views)
- **Role:** Thin UIs for CSP operators and participants to support onboarding, catalog publishing, and transfers.
- **February:** Minimum viable feature set, functional but not product-grade.

### 9. Industrial Data Plane & OPC UA Cloud Library
- **Role:** Industrial data plane using OPC UA semantics via the UA Cloud Library.
- **February:** End-to-end demo possible but scenario-specific and still prototype-level.

---

## 3. February 2026 Madrid Demo – Readiness Snapshot
The demo will be given at the Data Space Symposium 10th & 11th of February in Madrid.
https://www.data-spaces-symposium.eu/

### Scenario Focus
- Certificate-driven trusted data sharing in a supply-chain context.
- Protocols: DSP for catalog, contract and transfer, DCP for verifiable claims.
- DPS present in prototype form alongside the classic data plane.

### What Will Be Demonstrated
- Automated tenant onboarding via CFM.
- Wallet provisioning with membership and ISO certificate credentials.
- Publishing and discovering data offers via DSP.
- Contract negotiation based on credential-backed policies.
- Stable data transfer using the classic data plane under new control logic.

### Key Risks and Constraints
- Data plane migration remains parallel to reduce demo risk.
- UI polish is intentionally limited.
- Industrial data plane integration should be clearly labeled as prototype.

## 4. Reference architecture docs
For deeper architecture details:
- Virtual connector architecture doc (in DCSA sources): https://github.com/Metaform/dcsa/blob/main/content/en/technical/virtual.connector.architecture.md
- CFM architecture notes: https://github.com/Metaform/connector-fabric-manager/blob/main/docs/developer/architecture/system.architecture.md
