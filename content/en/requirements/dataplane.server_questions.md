# Architecture requirements

## System overview

![overview](overview.svg)

The data plane server is a thin application layer wrapped around a metadata index and three APIs:

- Dataplane Signaling: for communication with the control plane
- Ingest API: to make the data plane aware of new data and its authorization structure
- AAS API(s): allows consumers to get/download the data

A metadata index keeps track of the data structure and its authorization attributes.

This document contains questions, which - once answered - will serve as basis for technical requirements.

### Which AAS API do we need?

The AAS API Specification defines several [API groups](https://app.swaggerhub.com/apis/Plattform_i40/Entire-API-Collection/V3.1.1#/). Which ones are actually needed?
Assumption:

- AAS Registry API: to register new data sets upon ingest (provider side) and to browse (consumer side)
- AAS Repository API: to obtain AAS data and submodel identifiers
- Submodel Repository API: to obtain Submodel information + actual values

typical flow (?):

1. Registry stores descriptors (metadata + endpoints).
2. Consumer makes queries against AAS registry API → gets the AAS Descriptor.
3. Consumer follows descriptor endpoint (AAS Repository API) → fetches AAS.
4. Consumer resolves Submodel IDs from AAS
5. Consumer calls Submodel Repository API with IDs → gets the real data.

## Consumer-side APIs (AAS/Submodel APIs)

- The spec does not specify the filter expression notation. What filter queries do we need? Do people query by Shell-ID, Submodel/-ID? Some other property of the Submodel?
- Do we have to query by value, i.e. "give me submodels that match `'palletOfScrews.manufacturing_data.weight > 42_000'`"?
- Do people want to query for Submodel type, e.g. "give me all SoC values of all battery passports I have access to"?
- How many shells and submodels are realistic?
- Can shells/submodels change over time, i.e. can they be "patched"? The AAS API spec allows for that - is it done in practice?
- If patching is used: is the patch supplied by the shell's original owner, and if not: is the patch "pushed" to the original owner's shell (how?) or is it stored elsewhere and linked (how?)?
- Are shell tree always single-rooted? For example: Shell = `engine`, submodel = `proof_of_disassembly`. The `proof_of_disassembly` is typically supplied by another company. Can i query for the `proof_of_disassembly` "by submodel type/category" on the dismantler's AAS server and do I have to browse from the root (i.e. have to know that the root is an `engine`), or is the `proof_of_disassembly` represented as another shell?
- is the data provided in parts or in one fell swoop? i.e. is the AASX File Server API typically used?
- are any of the mutating API calls needed? patching shells, patching submodels, etc. (dismantler adds proof-of-destruction as submodel, a data value changes....)
- Assumption: submodel endpoint would then point to OPC UA server (specifically, a variable there), a file, a REST API, or anything similar?
- which AAS Server is being used in Catena-X?

## Ingest API

- how do providers supply the physical data? Files? Databases? APIs?
- Is there a common data format that the new data has to be in?
- Can we impose a unified data format or do we have to make format parsers extensible?
- Is the AAS Registry API used to register new data?
- is an uploaded single file an atomic unit (no introspection needed)?
- what is the relation NodeSets, Node values and <> AAS data?

## Authentication/Authorization

- which protocol (OAuth2, OIDC, mTLS,...) is used for authn/authz?
- what is the permission granularity, i.e. what roles/scopes are there? e.g.:
  - can view/modify a certain shell and all its submodels. easiest approach, would only require linking the AAS with [1..N] participants
  - can view/modify all submodels of a certain kind
  - can view/modify metadata of all shells
  - can view/modify metadata of all submodels, or of a particular kind
  - can view/modify shells of a kind
  - can view/modify data of a certain kind of submodels
- how are permissions/roles/scopes mapped onto data: is this a separate entry in the Ingest API body? is there going to be a separate UI?
- (authentication: could we use VCs via presentation flow? have the EDR (`DataAddress`) contain the token information? This would require an IdP Server with an authorization concept already in place: the IdP would need to "know" the participant and have roles/permissions attached to it.)

## Data upload scenarios

Which data upload scenarios do we have to consider? E.g.:

- drop single Excel sheet in OneDrive
- Install agent on-prem that periodically queries a data source (OPC UA) and pushes data into Ingest API
- Manual export button -> pushes into Ingest API
- more elaborate systems with TimeSeries DB, multiple sources, aggregators,...
