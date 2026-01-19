# Agricultural Data Standard (ADS) v1.0

**Status:** Public Draft  
**Author:** Harikrishnan V S  
**Version:** v0.1

This document is a public draft and is expected to evolve.

0. Preamble
Agriculture in India is increasingly driven by data — from soil sensors and weather stations to satellite imagery and farm management apps. However, today this data lives in silos. Each device vendor, platform, and program collects information in its own format, making it difficult to combine datasets, run large-scale analytics, or build reliable AI models.
The Agricultural Data Standard (ADS) v1.0 is proposed to solve this practical problem.
ADS defines a common, lightweight way for agricultural IoT data to move from the field to software systems, analytics platforms, and government programs. It is designed from the ground up for Indian farm conditions: low power devices, unreliable connectivity, and long deployment lifetimes.
ADS is company-led and open. It is not a regulation and does not replace existing systems. Instead, it provides a shared reference that device makers, agri-tech companies, researchers, and government bodies can adopt when interoperability and data reuse matter.
The long-term intent of ADS is to enable trusted data and model sharing across the agricultural ecosystem, with government platforms acting as neutral aggregators and distributors of insights rather than owners of proprietary systems.

1. Scope & Objectives
1.1 Scope
ADS v1 applies to:
Agricultural IoT devices deployed in farms, greenhouses, plantations, and allied sectors


Edge gateways aggregating data from field devices


Cloud platforms, analytics systems, and government data platforms consuming agricultural data


ADS v1 does not mandate specific hardware, vendors, or network technologies.
1.2 Objectives
ADS v1 aims to:
Enable interoperability across agro IoT devices and platforms


Ensure data is reliable, recoverable, and AI/ML-ready


Support operation in constrained rural environments


Reduce integration cost for governments and enterprises


Create a foundation for national-scale agricultural intelligence



2. Design Principles
ADS v1 is guided by the following principles:
Field-first: Designed for low power, low bandwidth, and unreliable networks


Interoperable: Vendor-neutral and open


Minimal: Defines only what is necessary for interoperability


Extensible: Supports evolution without breaking deployed systems


India-centric: Aligned with Indian agricultural and digital public infrastructure needs



3. Terminology & System Roles
ADS Device: A constrained field-deployed sensing or actuation unit


ADS Gateway: An intermediary system that aggregates, validates, and forwards device data


Upstream Systems: Cloud platforms, government systems, or analytics pipelines consuming ADS data


Field Registry: Canonical registry defining ADS data fields, units, and constraints



4. ADS v1 – Gateway Compliance Requirements
1. Purpose
This section defines the mandatory and optional requirements for any system claiming ADS v1 Gateway Compliance. A compliant gateway ensures reliable ingestion, normalization, buffering, and secure forwarding of agro IoT data from constrained devices to upstream platforms (cloud, government systems, AI/ML pipelines).

2. Gateway Role (Normative)
An ADS v1 Gateway SHALL:
Act as an intermediary between ADS-compliant devices and upstream systems


Terminate device-level protocols and expose internet-grade interfaces


Enforce ADS v1 data validation, versioning, and security policies


An ADS v1 Gateway SHALL NOT require direct internet connectivity from field devices.

3. Ingress (Device → Gateway) Requirements
3.1 Supported Data Format (Mandatory)
MUST accept CBOR (RFC 8949) payloads


MUST support numeric map keys as defined in the ADS Field Registry


MUST support definite-length maps and arrays only


MUST reject malformed or non-compliant CBOR payloads


3.2 Minimum Envelope (Mandatory)
Gateway MUST validate presence of the following fields:
device_id


timestamp (epoch seconds)


sequence_number


ads_version


sensor_payload


Payloads missing mandatory fields MUST be rejected or quarantined.
3.3 Transport Bindings (At least one required)
Gateway MUST support at least one of:
MQTT over TCP


MQTT-SN


HTTP (local network only)


LoRaWAN network server integration



4. Validation & Normalization
4.1 Schema Validation (Mandatory)
MUST validate payload against ADS v1 schema


MUST validate data types, ranges, and units


MUST tag invalid data with error codes (not silently drop)


4.2 Normalization (Mandatory)
MUST map numeric keys to canonical ADS field definitions


MUST normalize units where required (e.g., temperature to °C)


MUST attach gateway metadata:


gateway_id


received_timestamp


signal_quality (if available)



5. Buffering & Offline Behavior
5.1 Store-and-Forward (Mandatory)
MUST persist data locally during internet outages


MUST guarantee at-least-once delivery upstream


MUST preserve original timestamps


5.2 Retention (Minimum)
MUST buffer a minimum of 72 hours of data at nominal load


SHOULD support configurable retention policies



6. Egress (Gateway → Upstream) Requirements
6.1 Output Formats
Gateway MUST support:
JSON over HTTPS (mandatory)


Gateway SHOULD support:
NDJSON (streaming)


Apache Parquet (batch export)


6.2 API Compatibility
MUST expose REST or streaming APIs compliant with ADS v1 Cloud Interface


MUST support versioned endpoints (e.g., /v1/ingest)



7. Security Requirements
7.1 Device Trust
MUST authenticate devices using at least one of:


Pre-shared keys


Device certificates


Secure token exchange


7.2 Data Security
MUST encrypt all upstream communication (TLS 1.2+)


MUST not expose raw device credentials upstream


7.3 Isolation
MUST isolate device failures or compromises


MUST prevent lateral movement between devices



8. Control & Actuation (Optional but Standardized)
If actuation is supported, gateway:
MUST validate command authenticity


MUST support command acknowledgement


MUST log actuation events with timestamps and outcomes



9. Observability & Operations
Gateway SHOULD expose:
Health metrics (CPU, memory, storage)


Ingest rate and error counts


Last-seen timestamp per device


Logs SHOULD be exportable for audits and compliance.

10. Compliance Declaration
A gateway claiming ADS v1 compliance MUST:
Publish a compliance statement


Declare supported transports and features


Declare ADS version compatibility


Partial compliance MUST be explicitly stated.

11. Forward Compatibility
Gateways MUST accept newer ADS payload versions when backward-compatible


Unknown fields MUST be ignored, not rejected


This ensures long-term evolution without breaking deployed systems.

ADS v1 – Device Compliance Requirements
12. Device Role
An ADS v1 Device is a constrained field-deployed unit responsible for sensing and/or actuation. Devices prioritize power efficiency, reliability, and simplicity.
13. Device Data Format
MUST serialize payloads using CBOR (RFC 8949)


MUST use numeric keys defined in the ADS Field Registry


MUST include the ADS minimum envelope


MUST avoid floating-point values unless explicitly required


14. Device Envelope (Mandatory)
Each payload MUST include:
device_id (globally unique)


ads_version (e.g., 1.0)


timestamp (epoch seconds)


sequence_number (monotonic per device)


sensor_payload (numeric-key map)


15. Power & Reliability
Devices SHOULD batch readings when possible


Devices SHOULD tolerate intermittent connectivity


Devices MUST NOT require real-time acknowledgements



ADS v1 – Field Registry v1
16. Purpose
The ADS Field Registry defines canonical numeric keys, units, and constraints to ensure interoperability across vendors and regions.
17. Field Definition Structure
Each field definition includes:
Numeric Key


Name


Data Type


Unit


Valid Range


Description


18. Core Fields (Illustrative)
Key
Name
Type
Unit
Range
1
soil_moisture
uint
%
0–100
2
temperature
int
°C
-40–85
3
soil_ph
float
pH
3–10
4
humidity
uint
%
0–100

Registry updates MUST be backward compatible.

ADS v1 – Cloud & Government Ingestion Specification
19. Purpose
Defines how ADS-compliant data is ingested, stored, and consumed by cloud platforms, government systems, and AI/ML pipelines.
20. Ingestion Interfaces
MUST accept JSON over HTTPS


MUST support bulk ingestion


MUST preserve original device timestamps


21. Data Storage
Time-series databases recommended for raw data


Columnar formats (Parquet) recommended for analytics


22. AI/ML Readiness
Data MUST include location (when available)


Data MUST include quality and signal metadata


Missing data MUST be explicitly marked



ADS v1 – Certification & Compliance Levels
23. Certification Levels
Bronze
CBOR compliance


Minimum envelope


Basic transport support


Silver
Full schema validation


Offline buffering


Secure authentication


Gold
Advanced observability


Actuation support


Government ingestion compatibility


24. Compliance Process
Self-declaration


Automated test suite (recommended)


Periodic re-certification



ADS v1 – Governance & Stewardship
25. Stewardship and Ownership
ADS is currently stewarded by Harikrishnan V S, in an individual capacity.
At this stage, ADS is intentionally not owned by a legal entity, consortium, or government body. The goal of this phase is to allow open discussion, experimentation, and critique without institutional pressure.
The steward’s responsibilities are limited to:
Publishing and maintaining the ADS specification drafts


Curating feedback and proposals from the ecosystem


Releasing updated versions transparently


Future stewardship models (foundation, trust, or company-backed entity) may be proposed once there is demonstrated ecosystem adoption.

26. Change Management Process
Changes to ADS MAY be proposed by any ecosystem participant, including device manufacturers, agri-tech companies, researchers, NGOs, and government bodies.
26.1 Proposal Mechanisms
Change proposals MAY be submitted via:
A public email address managed by the ADS Steward


A feature request or issue submission on the official ADS website


Each proposal SHOULD include:
Description of the change


Motivation and use case


Backward compatibility considerations


The ADS Steward SHALL review proposals and publish decisions or draft updates transparently.

27. Government Participation Model
ADS is designed to work with government systems, not be controlled by them.
Government participation is envisioned in three non-controlling roles:
Consumer: Using ADS-compliant data for programs, dashboards, and policy insights


Auditor: Periodically reviewing data quality, lineage, and compliance claims


Contributor: Providing feedback, requirements, and datasets to improve the standard


Government agencies do not define ADS unilaterally and do not control versioning or release decisions.

28. Conflict Resolution
Conflicts related to ADS interpretation, implementation, or evolution may arise between ecosystem participants.
ADS v1 adopts a progressive conflict resolution approach:
Clarification Phase


The ADS Steward provides a written interpretation or clarification of the specification


Consultation Phase


Input is sought from affected stakeholders, including vendors and subject-matter experts


Specification Update or Guidance


If needed, a non-breaking clarification note or minor version update is issued


Independent Review (Optional)


For unresolved disputes, an independent advisory panel (technical and domain experts) MAY be consulted


This approach prioritizes continuity, backward compatibility, and ecosystem stability over rigid enforcement.

29. Versioning and Evolution
ADS follows semantic versioning: MAJOR.MINOR


Major versions introduce breaking changes


Minor versions introduce backward-compatible enhancements


The ADS Steward commits to maintaining backward compatibility within a major version wherever feasible.

ADS v1 – Summary
ADS v1 defines a practical, India-first, field-tested standard for agro IoT data. It prioritizes constrained devices, unreliable networks, interoperability, and AI readiness—enabling farmers, innovators, and governments to build a shared agricultural data foundation.

