# 1. Requirements for an industrial data plane server

<!-- TOC -->

- [1. Requirements for an industrial data plane server](#1-requirements-for-an-industrial-data-plane-server)
  - [1.1 Definition of terms](#11-definition-of-terms)
  - [1.2 Roles and components](#12-roles-and-components)
    - [1.2.1 Discovery Service](#121-discovery-service)
    - [1.2.2 Submodel Servers](#122-submodel-servers)
  - [1.3 Functional requirements](#13-functional-requirements)
    - [1.3.1 Public-facing APIs](#131-public-facing-apis)
      - [1.3.1.1 Query capabilities of public-facing APIs](#1311-query-capabilities-of-public-facing-apis)
        - [Query-by-ID](#query-by-id)
        - [Query by other properties](#query-by-other-properties)
      - [1.3.1.2 Mutating API calls](#1312-mutating-api-calls)
      - [1.3.1.3 Authentication and Authorization](#1313-authentication-and-authorization)
    - [1.3.2 Data manipulation and ingest (internal APIs)](#132-data-manipulation-and-ingest-internal-apis)
      - [1.3.2.1 Security, Authentication and Authorization](#1321-security-authentication-and-authorization)
    - [1.3.3 Dataplane Signaling API](#133-dataplane-signaling-api)
  - [1.4 Non-functional requirements](#14-non-functional-requirements)
    - [1.4.1 Modularity](#141-modularity)
    - [1.4.2 Scalability and Availability](#142-scalability-and-availability)
    - [1.4.3 Security](#143-security)
    - [1.4.4 Maintainability](#144-maintainability)
    <!-- TOC -->

An industrial plane server is a runtime component that allows data providers to serve their data to authorized
consumers. It is typically used in data spaces, where data providers and consumers do not know each other beforehand.

The term "industrial" indicates that the server is used in an industrial context, i.e., it has to deal with large
amounts of data, both high and low frequency of data changes. The domain is typically manufacturing, logistics,
industry, automotive, aerospace or similar. While concepts developed here may be transferable to other domains such as
healthcare, smart cities or finance, these are out of scope.

This document serves as a basis for any technical designs of such a data plane server.

## 1.1 Definition of terms

- data provider: an entity that owns data and wants to share it with others via a transmission channel.
- data consumer: an entity that wants to access data owned by a provider. The consumer typically accesses the data plane
  server's APIs to get the data. This could be REST API calls, file downloads, subscriptions to a streaming service or
  similar.
- data space: a virtual space where providers and consumers can share data in a trusted way
- control plane: a component that manages contract negotiation and transmission channels
- data plane: a component that serves data (or enables the serving of data) to authorized consumers
- authn/authz: "authentication and authorization," denotes the process of identifying a client and checking their
  permissions
- principal: an entity that can be authenticated, typically a user or a service, not necessarily a human actor
- asset: a physical thing that is identified by a unique identifier, its asset-ID

## 1.2 Roles and components

The data plane server is not one single application, instead it is a collection of components, each fulfilling a
different role.

These are:

### 1.2.1 Discovery Service

A single, central service that allows data consumers to find the AAS-ID and submodel identifiers for any given physical
asset, such as a serial number or a part number. The Discovery Service is the initial port-of-entry for all data
consumers. The Discovery Service may be comprised of multiple (logical) subcomponents and APIs.

To perform management operations, such as create, update, delete asset descriptor information, the Discovery Service
must expose an API that is intended to be used by internal actors (or other internal systems) only and that is not
exposed to the internet. In practice, this management API is implemented by the [AAS Registry API SSP-001](https://app.swaggerhub.com/apis/Plattform_i40/AssetAdministrationShellRegistryServiceSpecification/V3.0_SSP-001).

### 1.2.2 Submodel Servers

Submodels are containers of actual data in the AAS world, so each submodel server must implement the [Submodel Service API
SSP-003](https://app.swaggerhub.com/apis/Plattform_i40/SubmodelServiceSpecification/V3.0_SSP-003#/). There is no limit
how many submodel servers may exist per AAS, or what the cardinality of submodels per server is.

A submodel server (however implemented) is accessible from the internet and must implement appropriate authentication
and authorization mechanisms. It also must offer a way to manipulate the data it holds.

Each AAS must support multiple submodel servers, and each submodel server must be able to serve submodels for multiple AAS.

## 1.3 Functional requirements

### 1.3.1 Public-facing APIs

This chapter focuses on APIs that are exposed to other dataspace participants over the internet.

In many industrial contexts, the [Asset Administration Shell
(AAS)](https://www.plattform-i40.de/PI40/Redaktion/EN/Downloads/Publikation/aas-specification.pdf?__blob=publicationFile&v=6)
is the de-facto standard to represent digital twins of physical assets. While the AAS Specification defines a myriad of
APIs, typically only a subset is used in practice:

- [Discovery Service API
  SSP-001](https://app.swaggerhub.com/apis/Plattform_i40/DiscoveryServiceSpecification/V3.0_SSP-001): these endpoints
  allow resolving AAS identifiers for of physical assets, based on their serial numbers, part numbers, etc. It is
  typically used by data consumers to find the AAS identifier for a given physical asset. AAS of a physical asset they
  are interested in.
- [AAS Registry API
  SSP-002](https://app.swaggerhub.com/apis/Plattform_i40/AssetAdministrationShellRegistryServiceSpecification/V3.0_SSP-002#/):
  resolves the endpoint information for a given AAS and/or submodel identifier. Submodel information may be distributed
  across a company's IT landscape, so this API is needed to find the correct endpoint and protocol information.
- [Submodel Service API
  SSP-003](https://app.swaggerhub.com/apis/Plattform_i40/SubmodelServiceSpecification/V3.0_SSP-003#/): defines an API
  that every submodel endpoint, that serves actual data, must implement. Submodels are the actual containers of data in
  an AAS, and the systems that serve that data may be heterogeneous in nature, for example, an ERP system may implement
  the Submodel API and serve data from its database.

_Note: APIs for data consumers are not listed here._

#### 1.3.1.1 Query capabilities of public-facing APIs

In most scenarios the data consumers know the physical asset ID (e.g., serial number) and want to get data for that
asset. The primary query pattern is, therefore:

##### Query-by-ID

- query by AAS-ID (Shell-ID): the ID is specified in the request URL
- query by Submodel-ID: the ID is specified in the request URL
- query by SpecificAssetId: `specificAssetId` is a property of the AAS that contains information about the asset, for
  example, VIN numbers, owner information, etc.

> TODO: clarify how exactly the `specificAssetId` query works

##### Query by other properties

The APIs listed in [public-facing APIs](#131-public-facing-apis) do not define a query endpoint, but the AAS and
Submodel Repository API define a `/query/shells` and `/query/submodels` endpoint, respectively, that allow to query
shells and submodels using a custom query expression syntax.

> this requirement is OPTIONAL

#### 1.3.1.2 Mutating API calls

The AAS Specification defines several mutating API calls, for example, to patch shells or submodels or their respective
data. While industry use cases exist (e.g. [here and
ff.](https://eclipse-tractusx.github.io/docs-kits/kits/digital-twin-kit/software-development-view/interaction-patterns#3-updating-an-existing-submodel)),
their implementation would require an optional profile of the AAS Registry API
[SSP-001](https://app.swaggerhub.com/apis/Plattform_i40/AssetAdministrationShellRegistryServiceSpecification/V3.0_SSP-001).
This would also surface an entirely new set of requirements for [authn/authz](#1313-authentication-and-authorization).

> this requirement is OPTIONAL

#### 1.3.1.3 Authentication and Authorization

The data plane server shall enforce secure authentication and authorization mechanisms across all [public-facing
APIs](#131-public-facing-apis).

Clients making requests to these APIs must authenticate using industry-standard protocols such as OAuth2 or OpenID
Connect (OIDC). Authorization shall be implemented on a per-submodel basis. This means that authoring systems must be
able to specify access rules for each submodel, either for individual principals, for groups of principals or based on
roles/scopes.

**Example use case 1**: a data consumer, identified by their dataspace participant identifier (e.g., a
[DID](https://www.w3.org/TR/did-core/)), may be authorized to discover only a certain subset of all available AAS, and
within those AAS, they may only be authorized to access submodels of a certain type (e.g. "battery passport" submodels).

If [mutating API calls](#1312-mutating-api-calls) are implemented, the data plane server must also enforce authorization
on PUT, POST, PATCH and DELETE calls, and the authorization backend must be able to resolve permissions for these calls.

**Example use case 2**: a dataspace participant contributes a new submodel to an existing AAS, e.g., a maintenance
report for a turbine. This means the turbine manufacturer's AAS is _patched_ by adding a new `SubmodelDescriptor` to the
AAS server, which would point to the maintenance provider's submodel endpoint. The turbine manufacturer must authorize
the maintenance provider to execute mutating calls on their AAS.

### 1.3.2 Data manipulation and ingest (internal APIs)

This chapter focuses on systems and APIs that are intended to be used by internal actors (or other internal systems)
only. These systems and APIs are not exposed to the data space and should be protected by network-level security
mechanisms such as firewalls or VPNs. In addition, they may require a different authentication and authorization
mechanism than the [public-facing APIs](#131-public-facing-apis). Here, conventional enterprise mechanisms such as
Active Directory, LDAP or SAML may be used to identify authorized actors.

The following operations must be implemented on the dataplane server:

- _adding new assets_: the system shall offer a possibility to register new assets with the dataplane server's discovery
  service. This could be implemented using the [Discovery Service API
  V3.1.1-SSP-001](https://app.swaggerhub.com/apis/Plattform_i40/DiscoveryServiceSpecification/V3.1.1_SSP-001)
- _changing asset information_: the system shall offer a possibility to change asset information, such as adding new
  submodels or changing existing ones, or manipulate other AAS metadata. This could be implemented using the [AAS
  Registry API
  3.1.1_SSP–002](https://app.swaggerhub.com/apis/Plattform_i40/AssetAdministrationShellRegistryServiceSpecification/V3.1.1_SSP-002)
- _changing submodel data_: submodels hold the actual data, so it must be possible to update them. This could be
  implemented using the [Submodel Service API
  V3.1.1_SSP-001](https://app.swaggerhub.com/apis/Plattform_i40/SubmodelServiceSpecification/V3.1.1_SSP-001).

Please note that using those AAS APIs is **not mandatory** to fulfill the requirements listed above. It is entirely
possible to implement the "changing submodel data" requirement by other means, for example, by directly manipulating an
ERP system or uploading static files to a system. The speeds-and-feeds of the data must be taken into account here.

Similarly, to fulfill the "adding new assets" requirement, the Discovery API ([see Public-facing
APIs](#131-public-facing-apis)) may simply point to a flat file or database that contains AAS information, and
modifications to it happen via file/db access. Here, the level of concurrency and the frequency of those updates will
play a significant role.

> **Open question**: what ingest formats do we have to consider? Requirements were brought forward to support OPC UA
> Nodeset files as well as ERP systems being the actual data source. Are there other formats such as CSV files? JSON
> files containing AAS-compliant submodel data?

#### 1.3.2.1 Security, Authentication and Authorization

All internal APIs must be protected with appropriate authentication and authorization mechanisms. Which exact mechanisms
are used is dependent on the system(s) used to fulfill the requirements listed above.

There are, however, some generic requirements that all those mechanisms and systems **must** fulfill. Systems to
manipulate data **must**:

- be deployable in different security realms from the public-facing APIs. Security mechanisms (OAuth2, ...) **may** be
  different, but identity realms and security principals **must** be different. It should not be possible to use the
  same principal to authenticate against public-facing APIs and internal APIs to curtail the risk of privilege
  escalation. In most cases, dataspace participant will likely even use different identity providers: users use their
  internal employee ID to authenticate for data manipulation, while dataspace participants use their participant ID to
  authenticate against the Discovery API etc.

- be deployable in different network segments (e.g., internal vs. external). This is to enforce the separation of
  network substacks such as webservers and context paths. Public facing APIs and internal facing APIs **must not** be
  served on the same web context.

- be deployable individually from public-facing APIs: to enable independent development and maintenance streams, the
  internal systems and APIs must be upgradeable independently of external APIs.

- multi-tenancy: all internal APIs and systems **must** be multi-participant capable. For example, in order to update an
  AAS, the system must be able to identify the participant, verify that that participant owns the AAS, and then verify,
  that the user has the appropriate permissions for an UPDATE operation. Here, the term "participant" refers to a
  dataspace participant, not an internal user. A tenant (a company) may have multiple participants, and each participant
  may operate multiple data plane servers.

See also the [non-functional requirements](#14-non-functional-requirements) section of this document.

### 1.3.3 Dataplane Signaling API

Data plane server implementations **must** implement the [Dataplane Signaling
API](https://github.com/Metaform/dataplane-signaling/blob/main/docs/signaling.md).

> Todo: clarify the exact registration requirements

### 1.3.4 No inbound reachability requirements

The data plane server **must not** impose any reachability requirements on internal services that would necessitate
inbound connectivity of those services from the internet. Services, that contain the physical data cannot be required to
be exposed to the internet, for example, by opening firewall ports or similar.

The main reasons are security, scalability and performance.

The data plane server **must** provide a mechanism to act as a reverse proxy or gateway for such services. In practice,
this means that in order to serve data, the data plane server must not (solely) rely on an internal service being
exposed to the internet, as would be the case with ERP systems or OPC UA servers that are typically not internet-facing.

A common use case in certain industries is to have the ERP system implement the Submodel Repository API and route this
endpoint to the internet. While some customers may still choose to do this, the data plane server **must** offer an
alternative way, i.e. a submodel server component.

In addition, this submodel server component **must** implement the [Submodel Service API
V3.1.1_SSP-001](https://app.swaggerhub.com/apis/Plattform_i40/SubmodelServiceSpecification/V3.1.1_SSP-001) to allow
the manual or automated ingestion of data from other systems.

## 1.4 Non-functional requirements

### 1.4.1 Modularity

The data plane server is not one single application, instead it is a collection of components that can be deployed,
maintained and managed independently. This is to enable different scaling and availability requirements for different
components as well as independent development and maintenance cycles.

The following components **must** support the following use cases:

- geo-replication and high-availability: the Discovery Service must be able to run on HA systems, e.g., in cloud
  systems, and must run as replicated instances in different regions to reduce latency and increase availability.

- low-latency: the Discovery Service must be able to respond to requests in a low-latency manner, as it is the initial
  port-of-entry for all data consumers.

- resource efficiency: all components must be able to run in a non-wasteful manner, i.e., always deploying a Discovery
  Service alongside a Submodel Server, even if not in use, is not acceptable.

Many deployments will likely use additional components, such as a management UI, an authorization backend or similar.
These components **should** be implemented as separate deployable units as well.

Concrete requirements for modularity are listed in subsequent sections.

### 1.4.2 Scalability and Availability

In most real-life deployments, the Discovery Service will be a standalone component to enable—among other
things—individual scaling and high availability. Requirements with regard to availability and latency are extremely high
because the Discovery Service is the initial port-of-entry for all data consumers.

Every consumer must hit the Discovery Service to resolve the AAS-ID for a given physical asset ID. While a
cascading/partitioned lookup of asset IDs is possible, the initial lookup must always be done against a root Discovery
service.

In addition, every new asset is registered with the Discovery Service, so the load on the Discovery Service is
significant if the number of assets is high and assets are added frequently. To further complicate matters, the
Discovery Service potentially receives asset registration requests from a large number of geographically distributed
data servers, so geo-replication and multi-region deployment are a must.

One way to achieve high scalability and availability is to implement the Discovery Service as a multistaged, cascading
(partitioned) service:

The system consists of a hierarchical tree of Discovery services, where the root Discovery Service only maintains a
table of asset-ID-to-Discovery-Service-URL mappings and delegates the actual lookup to the respective downstream
Discovery Service. This reduces the compute and DB load on the root Discovery Service (because the LuT can be easily
cached), improves the possibility of replication and geo-distribution, but likely increases the latency for the lookup
overall.

In addition, cascading/partitioned setups make the registration process more complex because each asset must be
registered with the root Discovery Service and then a second time with the respective downstream Discovery Service.

### 1.4.3 Security

While security is expected to be mostly homogenous across the public-facing APIs of the data plane server, the internal
APIs are likely to face more nuanced requirements and roles.

- Submodel servers: managed by a highly specialized group of administrators with deep domain knowledge. May not even
  require APIs to manipulate data. Instead, data is manipulated directly in the underlying systems (e.g., ERP systems).
  If a write-API is present, it must be highly protected. Submodel server operators must be able to also update the
  Discovery Service.

- Discovery Service: updated by submodel administrators and data stewards, who manage the overall data offerings in the
  form of AASes.

Generally speaking, the internal APIs are likely to experience a medium level of overall load and should be separated on
a network level, possibly from one another, but definitely from the public-facing APIs.

Public-facing APIs are likely to experience a high level of load and present a large attack surface, so they should be
fronted by hardened API gateways and load balancers with request-throttling and DDoS protection capabilities.

### 1.4.4 Maintainability

Modular and independently deployable components enable independent maintenance and development cycles and independent
teams with specialized knowledge. In addition, individual components are often provided by different vendors, for
example, ERP system vendors may provide submodel servers that integrate with their systems.
