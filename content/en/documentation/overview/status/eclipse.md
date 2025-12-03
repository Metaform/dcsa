**Eclipse Data Space Platform: Architecture and Project Status
Overview**

**Introduction and High-Level Summary**

The Eclipse Data Space platform is an open-source initiative to enable
secure, sovereign data sharing across organizations. It builds upon the
Eclipse Dataspace Components (EDC) framework and new complementary
projects to create a multi-tenant "data space as a service" platform.
Key capabilities include **automated participant onboarding**,
**federated identity and trust management**, and **decentralized data
exchange** aligned with International Data Spaces standards. Major
industry stakeholders -- including BMW, Catena-X, Microsoft, Airbus,
Decade-X -- are sponsoring and adopting this platform, with cloud
providers like Aruba and Opiquad preparing to offer managed services
based on it. This document provides an **architecture overview** of the
platform's components, their current status, and the **plan for the
February 2026 Madrid demo** aimed at showcasing an integrated solution
for real-world use cases (e.g. digital certificates in supply chains).

At a high level, the platform consists of:

-   A **Connector Fabric Manager (CFM)** for orchestrating multi-tenant
    connector instances and workflows.

-   The **Eclipse EDC "Virtual" Connector (EDC-V)** -- a next-generation
    connector control plane supporting virtualization (multi-tenancy)
    and decoupled data transfer.

-   The **EDC Identity Hub** (Wallet) for decentralized identity and
    verifiable credentials management.

-   The **"Classic" EDC Connector** (legacy control plane) for
    continuity with existing deployments.

-   **Cloud-specific Extensions** (Azure, AWS, Huawei) enabling
    connectors to leverage cloud services (storage, key management,
    etc.).

-   A new **Data Plane Signaling** specification and **Data Plane
    SDK/Core** for pluggable, high-performance data transfer mechanisms.

-   A **Dataspace Technology Compatibility Kit (TCK)** for compliance
    testing of data space protocols.

-   **User Interface components** for both participants and service
    providers (e.g. self-service portal for data providers/consumers and
    admin consoles for cloud operators).

All these pieces are being developed in tandem and will be integrated
for the upcoming **February 2026** demonstration. By that time, the goal
is to have an initial end-to-end solution demonstrator as alpha code
with minimal polish, showing automated onboarding of a company into a
data space, issuance of digital credentials, data catalog publishing,
contract negotiation, and secure data exchange between a provider and
consumer. The following sections elaborate each component and the
project's status, citing relevant source repositories and documents.

**Architecture and Key Components**

**Connector Fabric Manager (CFM)**

The **Connector Fabric Manager (CFM)** is the backbone that manages
tenants (organizations) and coordinates the provisioning of their
connector resources in a multi-tenant environment. In other words, CFM
is responsible for turning the platform into a
**"connector-as-a-service"** offering for many small or large
organizations under a cloud provider's domain. CFM maintains a **Tenant
Registry** and associated resources for each onboarded organization.
When a new company signs up, CFM creates a tenant entry (with an
identifier, e.g. domain name) and tracks where that tenant's components
are deployed (which cluster, nodes, etc.) and what resources have been
allocated.

Crucially, CFM enables **multi-tenancy**: rather than each company
manually deploying their own connector, the cloud service provider (CSP)
uses CFM to provision and manage connectors for them in a shared
infrastructure. This provides scalability and operational control -- the
CSP can see all tenant connectors and perform lifecycle operations via
CFM's console.

**Provisioning and Orchestration:** A major function of CFM is to
orchestrate all the steps required to onboard a tenant's connector. This
includes tasks like: reserving a DNS name for the new connector
endpoint, deploying a new **EDC-V connector instance** (or allocating
one in a multitenant pool), creating an Identity Hub wallet for the
organization, and populating initial credentials. These steps are
automated through a workflow engine in CFM -- effectively a provisioning
**orchestration** engine. The orchestration can handle asynchronous and
parallel tasks (e.g. waiting for DNS propagation, then creating the
wallet once the endpoint is known). Each cloud provider may have custom
steps (for example, Aruba's process for DNS might differ from
Microsoft's), so the CFM workflows are **extensible** per provider.
Out-of-the-box, the platform will offer a reference workflow for
demonstration, and CSPs can tailor it to their infrastructure needs.

In summary, CFM **"glues together"** the platform's components for each
tenant. It provides tenant lifecycle management (create/update/delete
tenants), keeps an inventory of the tenant's connector resources (the
running virtual connector, DNS, wallets, etc.), and triggers the
automated setup and configuration of those resources. By doing so, CFM
gives the solution its multi-tenant capabilities and greatly simplifies
onboarding -- from a tenant's perspective, they just click "Join
Dataspace", and behind the scenes CFM provisions all necessary
components.

***Status* GREEN.** The Connector Fabric Manager is under active
development in the
[Metaform/connector-fabric-manager](https://github.com/Metaform/connector-fabric-manager)
repository. It is currently in an **alpha stage**, with the basic tenant
management and provisioning workflow functionality being built. The goal
is to have CFM functional enough by February to onboard demo
participants automatically. CFM will also be delivered with a user
interface (see "User Interface" section) for CSP administrators to
manage organizations. After the demo, CFM is expected to enter a regular
6-8 week release cycle for iterative improvements aligned with updates
to EDC and other components. Goal is to land it in the Eclipse Github
platform soon early next year.

**Eclipse EDC "Virtual" Connector (EDC-V)**

The **Virtual Connector (EDC-V)** is the next-generation Eclipse
Dataspace Connector, redesigned for a multi-tenant, cloud-native
environment. It serves as the **Control Plane** for data exchange,
handling data space protocols, contract negotiation, and integration
with the identity & policy framework -- but importantly, **without
bundling a fixed data plane**. In the new architecture, EDC-V will act
purely as the control logic ("broker") and rely on pluggable external
Data Plane services for actual data transfer. This is a shift from the
legacy EDC, which included an embedded data transfer runtime.

**Virtualization & Multi-Tenancy:** EDC-V is being engineered to support
multiple tenant connectors in a shared deployment. Each tenant
(organization) still has an isolated logical "connector" (with its own
identity, catalogs, and policies), but these may run co-located on
shared infrastructure. CFM interacts with EDC-V to create and configure
these virtual connectors for each new tenant. In practice, this could
mean spinning up dedicated connector instances per tenant in a
Kubernetes cluster, or potentially multi-tenant runtime instances -- the
exact mode is evolving. The platform will track each such **"virtual
EDC"** as a resource belonging to a tenant. By abstracting the connector
as a resource, CFM and EDC-V together enable dynamic scaling (adding
many small connectors on one cluster) and easier operations for service
providers.

**Decoupled Data Plane:** A major design decision is that EDC-V **no
longer incorporates a monolithic data transfer engine**. Instead, data
transfer is done by external Data Plane microservices that communicate
via a standard **Data Plane Signaling API** (see the Data Plane section
below). This change was motivated by the difficulty of one generic data
plane handling all use cases (HTTP downloads, streaming telemetry, large
file transfer, etc.). The new approach makes EDC-V more lightweight and
focused. EDC-V will implement the standard signaling protocol to
coordinate with any compliant data plane service. This allows
specialization: different industries or providers can plug in data plane
implementations optimized for their needs (e.g. high-speed file
transfer, MQTT streaming, etc.), while EDC-V remains the common control
logic. In essence, **EDC-V becomes a universal control plane** that can
work with various data plane services -- even potentially with other
connectors beyond EDC, as long as they speak the same protocols.

**Core Functions:** As the control plane, EDC-V handles all the familiar
functions of a data space connector except the raw data streaming. This
includes: participating in a **Federated Catalog** to advertise and
discover data offerings; performing **automated contract negotiation**
with counter-parties according to the Dataspace Protocol (DSP) and usage
policies; enforcing authorization and usage control policies; and
orchestrating the hand-off to a data plane when a transfer is initiated.
It also integrates with the Identity Hub for trust verification (e.g.
validating credentials of a partner connector) and with the
Registration/Onboarding services for initial trust bootstrapping.

***Status* GREEN**. The EDC Virtual Connector is being developed as a
separate codebase
([eclipse-edc/Virtual-Connector](https://github.com/eclipse-edc/Virtual-Connector)).
It is currently in **incubation**, with initial versions of the
multi-tenant control plane coming together. A preview of EDC-V is
expected to be part of the February demo. However, due to the
significant changes, the team is proceeding cautiously: for the February
milestone, the **legacy data plane will likely still be used** under the
hood for stability, with the new signaling-based data planes to be
integrated later in 2026 once fully tested. In other words, EDC-V's
control features will be shown by February, but the switch to the new
data plane architecture will follow afterwards. The existing EDC code
has been refactored to separate data plane concerns, and by demo time
EDC-V should operate as a control plane that can manage data transfers
via the old mechanism, demonstrating the concept. Post-February, work
will intensify on fully adopting the new data plane SDKs and optimizing
multi-tenant performance.

**Identity Hub (Wallet Service)**

The **EDC Identity Hub** is the component responsible for managing
organizational identities, **decentralized identifiers (DIDs)**, and
**verifiable credentials** in the data space. Each participant in the
data space (each company/tenant) is provided with an Identity Hub
instance -- essentially their **"wallet"** for storing and presenting
credentials. The Identity Hub ensures that every connector has an
associated identity anchored in a trust framework, which is critical for
secure and trusted data exchange.

Key functions of the Identity Hub include:

-   **DID management:** creation and resolution of a participant's
    decentralized identity (for example, using DID:web or other DID
    methods). This DID serves as the connector's identity in protocol
    handshakes.

-   **Verifiable Credential storage:** The hub can hold credentials such
    as organization attestations, membership proofs (e.g. membership in
    Catena-X or another alliance), certifications (ISO quality
    certificates, etc.), and usage policies. These credentials can be
    presented to other parties or used to satisfy policy checks during
    contract negotiation.

-   **Identity Provider interface:** The Identity Hub can interface with
    external authorities (e.g. a **Registration Service** or trust
    framework) to receive signed credentials. For instance, when a new
    company joins the ecosystem, a registration authority might issue
    them a **Dataspace Membership credential** which is stored in their
    hub. It also interacts with the **Trust Services** (like DAPS in IDS
    or equivalent in DSP) as needed to manage authentication tokens.

In the reference architecture of EDC, the Identity Hub is listed as a
core component ensuring secure access and identity management. It
provides each participant with control over their identity data in line
with data sovereignty principles. During onboarding, after CFM creates a
wallet for the tenant, initial credentials (such as a dataspace
membership or a participant ID) are populated into the Identity Hub.
Later, if the participant obtains other credentials (for example, **an
ISO compliance certificate issued by an authority**), those can be added
to the Identity Hub as verifiable credentials. These credentials become
usable in data sharing scenarios -- e.g. automatically proving to a data
consumer that the data provider is certified for certain standards.

***\
Status* GREEN**. The Identity Hub is developed under
[eclipse-edc/IdentityHub](https://github.com/eclipse-edc/IdentityHub).
It has been a part of the Catena-X reference implementations and is
relatively **mature** in concept. The current implementation uses DID
web and supports storing credentials and serving identity documents. For
the February demo, the **"certificates" use case is central**, meaning
the Identity Hub will be used to store a company's ISO certificate as a
verifiable credential and share it as part of a data exchange flow. The
team is ensuring the Identity Hub integrates seamlessly with EDC-V and
CFM by demo time -- e.g. automated creation of DIDs and loading of
sample credentials during provisioning. Beyond February, Identity Hub
development will focus on robust DID integration (potentially supporting
other DID methods), enhanced credential governance, and integration with
onboarding flows (to eliminate today's largely manual onboarding process
in ecosystems like Catena-X).

**Legacy "Classic" EDC Connector**

The **Classic EDC Connector** refers to the existing Eclipse Dataspace
Connector (EDC) implementation that has been used in production pilots
(e.g. Catena-X) up to now. It is essentially the **monolithic version**
of the connector combining control plane and a default embedded data
plane in one runtime. This legacy connector (found in
eclipse-edc/Connector) provides core services such as contract
negotiation, policy enforcement, IDS/DS Protocol implementations, and a
basic HTTP file transfer data plane.

While the new architecture moves toward EDC-V and external data planes,
the classic connector remains important for two reasons:

1.  **Stability and Backward Compatibility:** The classic connector is
    proven in existing use cases. For the initial phase (until the new
    architecture is fully hardened), it will continue to be used in
    parallel. In fact, for the February demo, the team decided to
    **retain the old data plane internally to reduce risk** --
    essentially running the new control logic with the old transfer
    mechanism for now. This ensures that the demo environment is stable
    and avoids last-minute issues from completely new components.
    Stakeholders who have existing EDC deployments can rest assured that
    those will not be suddenly obsolete; the classic connector will be
    supported (labeled "Classic EDC") during the transition period.

2.  **Feature Parity and Transition:** The legacy EDC already includes
    features like a basic **Federated Catalog**, asset index, policy
    engine, etc. The new Virtual Connector must provide equivalent or
    improved functionality. Until EDC-V covers all these, the classic
    EDC codebase serves as a reference. The development of EDC-V draws
    from the classic's core modules (many will be reused or slightly
    refactored). The classic connector's extensions (for cloud,
    protocols, etc.) are also being migrated. So, the "Classic" remains
    in use as we incrementally transition to the new modules.

In presentations, this component may be referred to as **"EDC Classic"**
or **"Legacy Connector"**, and color-coded separately. Functionally, it
includes control+data plane integrated, and uses the older IDS
Information Model and protocols (IDSCP v2, or RESTful DS Protocol) as
implemented in current EDC releases.

***Status*** **GREEN.** The Eclipse Dataspace Connector project is
active and currently at version 0.15.0 as of Dec 2, 2025, see
[eclipse-edc/Connector](https://github.com/eclipse-edc/Connector). It's
incubating towards a 1.0 release with the new architecture. No major new
features are expected on the legacy architecture; effort is focused on
carving out the new control plane and aligning with the DSP standard.
The classic connector is **stable** and has a broad set of extensions
(Azure, AWS, etc.) which will continue to be used. During Q1 2026, we
anticipate possibly labeling the classic connector as "maintenance
mode", once EDC-V proves itself. However, for safety, the February
demonstration will leverage the classic connector's battle-tested data
exchange capabilities behind the scenes. Stakeholders currently running
EDC 0.x connectors (e.g. in Catena-X) will have a smooth upgrade path:
they can continue with classic connectors short-term, and gradually move
to the Virtual Connector and new components as those reach production
quality.

**Cloud Technology Extensions (Azure, AWS, Huawei)**

To support deployment on different cloud platforms and to integrate with
cloud-native services, the project includes **Technology-specific
extension modules** for Azure, AWS, and Huawei Cloud. These are separate
repositories under Eclipse EDC (e.g., eclipse-edc/Technology-Azure,
eclipse-edc/Technology-Aws, eclipse-edc/Technology-HuaweiCloud). They
provide implementations of EDC's Service Provider Interfaces (SPIs) that
connect the connector to managed cloud services.

For example, the **EDC Technology Azure** extensions include:

-   Storage and state management backed by Azure Cosmos DB (for things
    like the connector's asset index or transfer process store).

-   Secure vault integration using Azure Key Vault (storing credentials,
    keys, secrets).

-   Data transfer integration with Azure Blob Storage (enabling the
    connector to use a blob container as the data source/sink during
    transfers).

Likewise, the AWS extension likely provides integration with AWS S3 (for
data transfers and persistent storage) and AWS Secrets Manager or KMS
for key vault, and possibly DynamoDB for state. The Huawei Cloud
extension would map to Huawei's equivalents (e.g. OBS storage, KMS,
etc.).

These modules are essential for **cloud-ready connectors** -- instead of
using only in-memory or file-based defaults, a connector deployed in
production on Azure can use Cosmos DB and Key Vault as durable, secure
backends. This not only improves robustness and scalability, but also
simplifies compliance with enterprise policies by using cloud-managed
services.

All the cloud extensions plug into the connector via the extensions
mechanism. In fact, the EDC project structure explicitly separates core
from extensions to allow such cloud-specific implementations. As the EDC
README notes, the extensions layer is where "technology- and
cloud-specific code" resides -- e.g. a CosmosDB-based
TransferProcessStore, an Azure KeyVault-based vault, etc. This
separation ensures the core remains cloud-agnostic, while users can
assemble a connector with the extensions suited to their environmentg.

***Status:* GREEN*:*** The cloud technology extensions are relatively
**well-developed** and have seen releases alongside EDC core (e.g. Azure
extension version 0.14.1 was released Oct 2025), see the 3 repositories
here [eclipse-edc
repositories](https://github.com/orgs/eclipse-edc/repositories). They
are being continuously updated to stay compatible with the evolving
core. By February's integration demo, we expect the Azure, AWS, and
Huawei integrations to be functional, enabling deployment on multiple
clouds. Microsoft has contributed significantly to the Azure module
(evidenced by official Azure documentation referencing the EDC Azure
deployment), and other community members contributed AWS and Huawei
support. These modules will be utilized during the demo if the scenario
involves multi-cloud operation (for instance, one participant's
connector on Azure and another's on Aruba Cloud using Huawei extension).
Aruba, Opiquad, and other CSP partners are also building on these to
create their **Dataspace-as-a-Service** offerings; for example, Aruba's
environment uses the CFM with underlying connectors presumably
leveraging these extensions for their cloud stack.

**Data Plane Signaling and Data Plane Core (SDKs)**

In the new architecture, **data plane** functionality is being spun out
into its own set of projects to allow greater flexibility and
performance. There are two closely related efforts: the **Eclipse Data
Plane Signaling** specification (protocol) and the **Eclipse Data Plane
Core/SDK** implementations.

**Data Plane Signaling (DPS):** This is a standard protocol that defines
how the control plane (EDC or any connector) interacts with a data plane
service. It covers the control messages to initialize a data transfer,
establish connections, coordinate streaming, etc. The idea is to have a
unified signaling spec so that different data plane implementations can
all hook into the connector in the same way. This is crucial for
interoperability -- if a connector supports DPS, it could drive any
compatible data plane, and theoretically, any connector (not just EDC)
that speaks DPS could use any DPS-enabled data plane. The Data Plane
Signaling project (see eclipse-dataplane-signaling) is defining these
protocols, likely aligned with upcoming **IDSA/GAIA-X** standards for
secure data transfer.

**Data Plane Core & SDKs:** Given the DPS spec, the Data Plane Core
project (eclipse-dataplane-core) provides libraries and SDKs to help
implement actual data plane services. Essentially, it offers the common
"building blocks" for constructing a data plane that can register with a
connector, handle transfer requests, and enforce the data agreement
(like usage policies) at the data stream level. According to the team,
the data plane core contains SDKs (potentially in multiple languages)
that implement the signaling and some base data transfer logic. For
example, a Java library to quickly create a data plane microservice that
supports HTTP download, or a Go library for a high-throughput file
pipeline -- all following the standard contract.

The **purpose** is to enable an ecosystem of specialized data planes:
*"each data transfer may have different requirements"* -- one might need
streaming telemetry, another bulk transfer, another asynchronous API --
so instead of one engine trying to do all, various fit-for-purpose data
planes can be built. The data plane core will likely ship with one
**generic HTTP data plane** as a reference (to ensure there's a default
available). But others (e.g. an **"industrial data plane"** optimized
for manufacturing IoT, or a **high-security enclave data plane** for
confidential computing) can be developed by third parties using the SDK.
The term "Industrial Data Plane" was mentioned as a category possibly
for specialized needs (the transcript alludes to an *"industrial data
plane"* as one of the big buckets), which presumably refers to a
specific data plane implementation optimized for industry scenarios.

**Connector perspective:** With DPS in place, the EDC-V connector will
solely act as a client to this API. When a contract is agreed and it's
time to transfer data, EDC-V will invoke the Data Plane API to either
send or receive data. The data plane service then actually streams the
data between source and destination endpoints, enforcing any usage
control if needed (e.g. converting or filtering data according to
policies). EDC-V no longer needs to understand the details of data
streaming -- it delegates that to the external service and just monitors
the outcome via signaling callbacks.

***\
Status* GREEN*:*** The Data Plane Signaling project has been approved at
Eclipse and is in progress, see
[eclipse-dataplan-signaling/dataplane-signaling](https://github.com/eclipse-dataplane-signaling/dataplane-signaling).
The Data Plane Core/SDK is also incubating. By February 2026, the
**signaling specification will be near finalized** (to support
integration testing), and an initial SDK and reference data plane should
be available. However, as noted, the team made a tactical decision **not
to switch the February demo connectors to the new DPS-based data plane
yet**, to avoid "too much churn" at once. The old embedded data transfer
will be used just for the demo to reduce risk. That said, we expect to
show at least a **prototype of a new data plane service** in February,
possibly running in parallel or as an optional feature. The goal is
after the demo, EDC-V will drop the legacy transfer code and fully adopt
the DPS approach. We anticipate multiple organizations contributing data
plane implementations in 2026. The **Eclipse Dataspace TCK** (below)
will also test the compliance of both connectors and data planes to the
signaling and protocol standards, ensuring everything works together.

**Eclipse Dataspace TCK (Compliance Test Kit)**

To foster interoperability and confidence in the ecosystem, the project
includes the **Eclipse Dataspace TCK (Technology Compatibility Kit)**.
The Dataspace TCK is essentially a suite of automated tests that verify
conformance of connectors and related components to the relevant data
space protocols and standards. Given that multiple vendors and
open-source projects (e.g. Eclipse EDC, Catena-X connectors, Dataspace
Connector from Fraunhofer, etc.) exist, a common TCK helps ensure they
can all work together in a data space.

Key points about the TCK:

-   It is designed to be **modular and extensible**. The core framework
    is in Java (built on JUnit) but it can test any runtime, even those
    implemented in other languages, by exercising their [external
    APIs](https://projects.eclipse.org/projects/technology.dataspacetck#:~:text=The%20Eclipse%20Dataspace%20TCK%20will,pipeline%2C%20or%20an%20online%20service).
    The TCK can be run from command line, CI pipelines, or as an online
    service.

-   The first focus is on [the **Dataspace Protocol
    (DSP)**](https://docs.internationaldataspaces.org/ids-knowledgebase/dataspace-protocol#:~:text=Dataspace%20Protocol%202024,planned%20as%20compliant%20implementation)
    compliance. DSP is the new specification (from IDSA) for connector
    interaction, which covers things like how two connectors negotiate
    and transfer data. The TCK will validate that all "normative
    statements" in the DSP spec are correctly implemented by a
    connector. For example, it will test that a connector properly
    enforces contract offers, that it can process and send
    self-descriptions, handle catalog queries, etc.

-   Over time, the TCK can be extended to other standards: e.g. checking
    compatibility with **IDS Information Model**, **Verifiable
    Credentials exchange**, or future protocols.

-   It provides an **independent verification** mechanism. Implementers
    can run the TCK against their software to identify any deviations
    early. Eventually, passing the TCK could be a prerequisite for
    official certification in ecosystems like Catena-X or IDSA.

***Status** **GREEN**:* The Eclipse Dataspace TCK project is in
**Incubating** phase at
[Eclipse](https://projects.eclipse.org/projects/technology.dataspacetck#:~:text=and%20to%20provide%20an%20independent,mechanism%20to%20validate%20conformance)
and [eclipse-dataspacetck](https://github.com/eclipse-dataspacetck). An
alpha version (focused on DSP) has been released recently. By the time
of the demo, the TCK will likely be in use internally to ensure the new
EDC components meet the DSP standard. In presentations to stakeholders,
it can be noted that **the platform aims to be fully compliant** and
that an independent TCK is in place to guarantee this. After February,
as the platform approaches production, we expect the TCK to mature and
perhaps a **certification program** to emerge (especially for ecosystem
partners or third-party connectors wanting to join the data space
network). This is important for stakeholders like BMW and Airbus, who
require that any connector in their supply chain's data space adheres to
the agreed standards -- the TCK is the tool that provides that
assurance.

**User Interfaces (Portals and Dashboards)**

While much of the focus is on back-end services, the project is also
developing the necessary **User Interface (UI)** components to make the
platform accessible to users. There are two main categories of UIs:

-   **Participant Dashboard:** A web UI for data space participants
    (companies) to manage their own connector, data assets, and
    transactions. In earlier Catena-X implementations, this role was
    served by the "Data Dashboard" provided by Fraunhofer. In the new
    platform, this will be realized as part of CFM's front-end or a
    standalone portal where an organization can log in to see their data
    offers, subscriptions, view catalog, and monitor transfers.
    Essentially, it's the **self-service portal** for a tenant. This UI
    will incorporate functions like uploading data asset metadata,
    triggering data sharing workflows, viewing credential status (e.g.
    what certificates the company has in its wallet), etc.

-   **CSP Management Console:** A UI for the **Cloud Service
    Provider/Operator** to manage the multi-tenant environment. Through
    this console, an operator (for example, Aruba's admin team) can
    onboard new organizations, trigger or schedule provisioning tasks,
    monitor all tenant connectors' health, and possibly manage
    infrastructure settings (clusters, scaling, etc.). It provides an
    overview of the whole "fabric".

The development of these UIs is a collaborative effort. **Aruba** has
already built an initial management UI as part of its early adoption of
CFM. In fact, Aruba's codebase for their PoC UI is being contributed as
the starting point for the open-source UI. This UI will be further
developed and "cleaned up" for inclusion in the platform. The plan is to
**white-label** it such that other CSPs can use it -- Aruba indicated
that a CSP could take the Aruba solution and rebrand it as their own
data space service.

Additionally, **Fraunhofer** (and possibly others like BMW's IT team)
are contributing UX knowledge from the prior Data Dashboard. The idea is
to merge the best elements of Aruba's UI and Fraunhofer's dashboard
design into a cohesive interface within CFM. The CFM UI will thus have
sections for both the provider (CSP) and the tenant (organization user).
It may be architected as two apps (one for CSP, one for participants),
or a single app with role-based views -- to be determined.

Key features expected in the UI by February include: basic tenant
registration forms, a view of the provisioning workflow status for a new
onboarding (so the operator can see if DNS, connector, wallet creation
succeeded), and a simple data offer publishing screen for the
participant. Given the tight timeline, the UI might still be rudimentary
by the demo, but there is recognition that **"we need the UI"** to be
usable soon, as it's the primary touchpoint for many stakeholders.

***Status ORANGE**:* The UI components are currently under development
in private repositories to be open-sourced. Aruba's is going to
contribute UI code soon to be reviewed by the Eclipse project. By the
Madrid session in Feb 2026, we aim to show a working portal: for
example, a demo where a new company is onboarded through a web form,
then that company uses the dashboard to publish a data asset and the
consumer uses their dashboard to find and request it. Fraunhofer's team
is actively working on this integration. Post-demo, making the UI more
feature-rich and user-friendly will be a priority (stakeholders will
expect a polished interface as the platform matures). This includes
functionalities like audit logs, billing information for the service
(for CSPs), advanced data catalog search, and integration of identity
credential management (e.g. a screen to upload or view the company's
certificates in the Identity Hub).

**TODO:** Repository Links? [\@Jim
Marino](mailto:jmarino@metaformsystems.com)?

**Industrial Data Plane and OPC UA Cloud Library**

The Industrial Data Plane is a domain-specific implementation of a data
plane designed for industrial automation and manufacturing data
exchange. It leverages the OPC UA Cloud Library developed by the [OPC
Foundation](https://opcfoundation.org/) as a foundational building block
for semantic data modeling, machine-to-machine interoperability, and
metadata federation across manufacturing participants. This integration
ensures compatibility with existing shop floor systems and IIoT
platforms, aligning the Eclipse Dataspace ecosystem with industrial data
standards.

**Role in Eclipse Dataspace Architecture**

Within the broader Eclipse Dataspace platform:

-   The Industrial Data Plane functions as a pluggable data transfer
    component, compliant with the emerging [Data Plane Signaling
    Specification](https://github.com/eclipse-dataplane-signaling/dataplane-signaling),
    allowing connectors like EDC-V to coordinate secure transfers of
    industrial data from OPC UA-compliant sources.

-   It enables semantic-rich data exchange, leveraging the OPC UA
    information model to ensure consistent interpretation of machine
    data, telemetry, and metadata across participants.

-   This plane is especially relevant for manufacturing use cases that
    require high-frequency, structured telemetry sharing (e.g.,
    predictive maintenance, digital twins, regulatory compliance).

**OPC UA Cloud Library: Milestone and Current Status**

The [OPC UA Cloud
Library](https://github.com/OPCFoundation/UA-CloudLibrary/) is a central
public repository for publishing and discovering OPC UA information
models via RESTful APIs and JSON payloads.

-   **Milestone 2** of the project (see [Milestone #2 on
    GitHub](https://github.com/OPCFoundation/UA-CloudLibrary/milestone/2))
    is currently active and focuses on:

    -   Enhancing multi-tenancy and onboarding mechanisms.

    -   Extending support for dynamic model discovery and validation,
        crucial for data space connectors to resolve metadata at
        runtime.

    -   Improving alignment with cloud-native identity and access
        mechanisms, making it easier to bind credentialed access to
        industrial endpoints.

This milestone is directly relevant for the February 2026 Madrid
demonstration, as it will showcase how small manufacturers using OPC
UA-enabled devices can expose their asset metadata and sensor data
through the Cloud Library, and then make it consumable by data space
connectors in a policy-compliant, credentialed manner.

**Integration Plans for the February 2026 Demo**

For the Madrid session:

-   The goal is to demonstrate an end-to-end scenario where an OPC UA
    server's metadata and data (e.g., ISO certificate info, production
    metrics) is published to the Cloud Library.

-   An EDC-V connector (deployed via the Connector Fabric Manager) will
    query the Cloud Library, match the model with its consumer's
    requirements, and initiate a data plane session using the Industrial
    Data Plane plugin.

-   The data will flow under enforceable usage policies, backed by
    verifiable credentials from the IdentityHub.

-   This validates that **small industrial players** (via CSP/MSP
    onboarding) can **participate in sovereign data spaces** using
    existing OPC UA infrastructure.

**Strategic Importance**

This integration:

-   Enables **brownfield compatibility** with existing OPC UA-enabled
    factories and suppliers.

-   Provides a **standard path for onboarding industrial assets** into
    Eclipse data spaces, without needing to replace edge infrastructure.

-   Reinforces **semantic interoperability**, making it easier for data
    consumers (like OEMs or regulators) to reason about and trust
    incoming data.

**Repository and Resources**

UA Cloud Library GitHub:
<https://github.com/OPCFoundation/UA-CloudLibrary>

Milestone 2:
<https://github.com/OPCFoundation/UA-CloudLibrary/milestone/2>

Demo integration reference repo: Planned linkage with
[eclipse-dataplane-core](https://github.com/eclipse-dataplane-core)

**Status ORANGE:** The Industrial Data Plane is still in the early
stages of implementation. The OPC UA Cloud Library (Milestone 2) is
actively progressing and forms the semantic backbone for metadata
publication. For the February 2026 Madrid session, we aim to demonstrate
an end-to-end flow: a small manufacturer publishes metadata (e.g. ISO
certificates) from their OPC UA server into the Cloud Library; an EDC-V
instance (provisioned via CFM) queries this metadata and initiates a
policy-based transfer via the industrial data plane. This will validate
how brownfield industrial participants can be onboarded through a CSP
and start sharing standardized data securely. While the library and
signaling APIs exist, there is no dedicated "industrial SDK" yet---the
February demo will use static or prototype flows. Post-demo, the focus
will shift toward integrating dynamic discovery, credential-bound access
to live data, and SDK extensions within
[eclipse-dataplane-core](https://github.com/eclipse-dataplane-core) to
support real-time, semantically-enriched telemetry exchange for IIoT use
cases.

**Plans for the February 2026 Madrid Demo**

The February session in Madrid is a crucial milestone where the team
will demonstrate an integrated end-to-end scenario to the consortium
stakeholders. The demo is expected to show the platform's capabilities
in a realistic use case, namely **Certificate-based data sharing in an
automotive supply chain context** (as identified by Catena-X and
others). Here's what is planned and how each component plays a part:

-   **Onboarding Automation:** The demo will start by showing how a new
    company (e.g., a small parts supplier) can onboard seamlessly.
    Instead of the current manual process (filling forms and waiting for
    an operator to register them in some database, as is done via
    Cofinity-X today), the platform will automate this. Using the CFM
    UI, an operator (or the company itself) initiates onboarding. CFM
    then provisions the tenant: reserves a DNS/web address for their
    connector, deploys their EDC-V instance, and creates their wallet,
    all in one go. The demo will highlight that what used to take phone
    calls and paperwork now happens in minutes through an automated
    workflow.

-   **Identity & Trust Establishment:** As part of onboarding, the
    company receives a **verifiable credential** asserting, for example,
    "Catena-X Membership" or some equivalent trust onboarding token,
    which is stored in their Identity Hub. Additionally, the primary
    **use case credential -- an ISO certificate** (for instance ISO 9001
    quality management certification) will be loaded into their Identity
    Hub. The demo scenario assumes that sharing this certificate down
    the supply chain is important (e.g. an OEM like BMW wants to ensure
    all sub-suppliers are certified). This certificate will be
    represented as a verifiable credential in W3C format. The Identity
    Hub's role will be shown by perhaps pulling up the credential
    (demonstrating the platform can store and later present it).

-   **Data Offering and Catalog:** The newly onboarded supplier will
    then use the Participant Dashboard to publish a **data offering**,
    for example, a "Digital Product Passport" or carbon footprint data
    for their part, which includes or references their ISO certificate.
    This will test the Federated Catalog functionality. The supplier's
    connector (EDC-V) will publish metadata about the asset/offer to its
    catalog endpoint, which can be discovered by others. In the demo, a
    consumer connector (e.g. OEM's connector) will query the catalog and
    find the supplier's offering. Because this is a controlled
    ecosystem, the catalog might not be fully open; it could be a
    **Federated Catalog** query via the dataspace protocol where only
    authorized participants (the OEM) see the listing.

-   **Policy Enforcement (Credential Use):** The exchange will be
    governed by usage policies that involve the certificate credential.
    For instance, the supplier's data product might have a policy: "Only
    share this data with partners who have a valid ISO9001 credential."
    The demo will illustrate how the OEM's connector presents its
    credentials during contract negotiation, and how the supplier's
    connector automatically verifies the OEM has the required
    certificate via the Identity Hub. Conversely, the OEM might require
    the supplier to have certain credentials. Since in this scenario the
    supplier is providing the certificate, it might actually be the
    **data** being shared itself. Alternatively, another policy could be
    shown such as "data can only be used if the consumer is a member of
    Catena-X" -- which again is checked via credentials.

-   **Automated Contract Negotiation:** Using the Dataspace Protocol
    (DSP), the two connectors will go through a **contract negotiation**
    without human intervention. The demo will highlight that once the
    consumer selects the data asset and clicks "request", the connectors
    automatically exchange offers and agreement, because everything
    (including checking the certificate) is encoded in the policies and
    credentials. This is a significant improvement in speed and security
    over traditional manual contract setups.

-   **Data Transfer:** After a contract is agreed, a secure data
    transfer will be executed. In February's demo, as noted, this will
    likely utilize the classic connector's data plane (e.g., an HTTP
    transfer or a background asynchronous transfer) rather than a
    brand-new mechanism, to ensure reliability. Nonetheless, it will
    appear seamless to the user. The data (perhaps a JSON file
    containing the product data and certificate) will move from the
    provider to the consumer. We will show in the UI that the transfer
    is completed successfully and perhaps that it obeyed constraints
    (for example, if there was a policy "delete after download", the
    consumer's connector enforces that after retrieving it).

-   **Monitoring & Dashboard:** The CSP's management console might be
    used in the demo to show how the operator can see the status of all
    connectors. For instance, they could show that the new tenant's
    connector is running on cluster X, that the workflow succeeded, and
    maybe drill into logs or metrics. The participant's dashboard would
    show the transaction history (e.g. "Data asset X shared with OEM Y
    at time Z under contract C"). This reinforces the transparency and
    auditability of the system.

The overall message for the Madrid session is that **"by February, all
of this stuff is going to be integrated"** -- the various previously
separate projects (CFM, EDC-V, Identity Hub, etc.) will function
together as a single solution. It will still be in an **alpha** state
(documentation may be sparse, not all edge cases handled), but it will
prove the architecture works. Jim (the chief architect) noted confidence
that *"we'll deliver something good for February... even if we have to
string it together"*. Indeed, contingency plans are in place (like using
the old data plane) to ensure something runs end-to-end by that time.

Importantly, the demo will also acknowledge what's **beyond February**.
The February milestone focuses on the **Certificates use case** (trust
and credential-based sharing) as the primary showcase. The team has
identified other use cases -- *Traceability* and *Tariffs* were
mentioned as next in line. While those won't be center stage in Madrid,
stakeholders such as Airbus (interested in traceability) or energy
partners (interested in tariffs/CO2 data) will be reassured that the
platform is built to accommodate those. The demo's modular architecture
(e.g. showing how adding a new credential or data type is
straightforward) will make that case.

Finally, the session will discuss the **roadmap** after the demo. This
includes completing the transition to the new Data Plane signaling
(since in February the system still uses the interim solution),
onboarding more partners into testing, and establishing a regular
release cadence. The idea of a synchronized 6-8 week release cycle
across projects was proposed -- meaning CFM, EDC-V, and others would
release in lockstep, so that new features arrive together and
dependencies remain aligned. This approach will be communicated to
stakeholders to set expectations for updates and to encourage their
teams to plan integration testing accordingly.

**Conclusion and Next Steps**

The Eclipse Data Space platform is on track to deliver a **unified,
multi-tenant data sharing solution** that meets the stringent
requirements of industrial data ecosystems like Catena-X. In summary,
the architecture features a robust cloud-agnostic manager (CFM) to
handle scale and automation, a revamped connector core (EDC-V) focused
on control plane tasks, and flexible identity and data transfer services
to ensure trust and performance. Key open-source repositories -- from
the main EDC components to cloud extensions and SDKs -- are actively
being developed and have reached critical milestones (many at version
0.x incubating toward stable 1.0 releases). The involvement of major
companies in development (Microsoft on Azure integration, Fraunhofer on
UIs, etc.) underscores the community momentum behind these components.

For stakeholders like **BMW and Airbus**, this platform promises easier
onboarding of your many partners and suppliers into data spaces, with
less manual IT effort and greater assurance of compliance (through
automated credential exchange and TCK-verified standards compliance).
**Catena-X members** will see the realization of the federated data
space vision with open-source connectors that can be deployed by any
participant or offered as a service by providers like **Aruba** (who,
along with **Opiquad**, intend to offer "Data Space as a Service"
leveraging this stack). The platform's cloud-neutral design (supporting
Azure, AWS, Huawei, on-premises etc.) means each company can choose
their preferred infrastructure without losing interoperability.

As we approach the Madrid demo in February 2026, all eyes are on
integrating the pieces and demonstrating value. The primary focus is the
**"credentials for trust"** use case -- ensuring that by demo day, a
small supplier can join a data network and instantly prove its
qualifications (like ISO certifications) and start sharing data under
controlled conditions. This is a compelling story for the sponsors,
addressing real pain points (onboarding friction and trust
verification). The demo will mark the platform's **alpha release**.

Post-demo, the project will move into a phase of hardening and
broadening: completing the data plane overhaul (so that by mid-2026 the
legacy data plane can be fully retired in favor of the new
signaling-based services), adding features for other use cases
(traceability across supply chain, dynamic pricing/tariffs for data,
etc.), and improving usability (more polished UIs, documentation, and
self-service). The **Eclipse Dataspace TCK** will likely reach a beta,
enabling early certification processes to begin, which will further
bolster confidence in adoption.

In conclusion, the project is progressing well and is aligned with the
timeline commitments made to stakeholders. By leveraging the cited
repositories and collaborative development, we are building a
sustainable open-source foundation for data spaces. The February Madrid
session will be an important opportunity to showcase the integrated
platform to BMW, Catena-X, Microsoft, Airbus, and others --
demonstrating not only technical readiness but also the **business
value** of faster partner onboarding, improved data sovereignty, and
cross-industry interoperability. We look forward to feedback and
continued support from all partners as we enter this next phase. The
source links and repositories included throughout this document provide
further technical details and will be useful for any team wishing to
dive deeper or even contribute to specific aspects of the platform.

**Architecture reads:**

[dcsa/content/en/technical/virtual.connector.architecture.md at main ·
Metaform/dcsa](https://github.com/Metaform/dcsa/blob/main/content/en/technical/virtual.connector.architecture.md)

[connector-fabric-manager/docs/developer/architecture at main ·
Metaform/connector-fabric-manager](https://github.com/Metaform/connector-fabric-manager/tree/main/docs/developer/architecture)
