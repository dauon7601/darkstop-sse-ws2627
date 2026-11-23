
# DarkStop – Requirements Document

Systems and Software Engineering I – WS 2026/27

Team:
- Da-Uon Kim
- Hafeezullah Haidari

Status: First Draft for Discussion
Date: November 26, 2026

## 1. Project Goal

DarkStop is a web application that recommends dark-sky observation
locations reachable by public transport in the RMV region.

The system considers sky darkness, cloud cover, walking distance,
and feasible outbound and return journeys.

## 2. Functional Requirements

### FR-01: Start Location and Travel Window
Priority: Must

Requirement:
The user shall be able to specify a starting location and a travel
time window.

Acceptance Criteria:
- A valid starting location and time window are accepted.
- Invalid or incomplete inputs are rejected with a clear message.
- A return deadline earlier than departure is rejected, unless
  the date explicitly indicates the following day.

### FR-06: Display of Recommendations
Priority: Must

Requirement:
The system shall display qualified destinations together with
sky quality, cloud cover, travel times, walking times,
planned outbound and return journeys, and snapshot information.

Acceptance Criteria:
- Every recommended destination displays the required information.
- Outbound and return connections can be inspected.
- The data snapshot is identifiable.
- Planned connections are not presented as guaranteed live journeys.

## 3. Non-Functional Requirements

### NFR-01: Reproducibility
Priority: Must

Requirement:
Identical inputs and the same data snapshot shall produce
identical results.

Acceptance Criterion:
Repeated requests with identical inputs and snapshot versions
produce identical results.

### NFR-03: Reproducible Execution
Priority: Must

Requirement:
The prototype shall run with bundled sample data using
Docker Compose, without requiring live third-party services.

Acceptance Criterion:
A clean test environment can start the prototype using
`docker compose up` and execute the agreed sample scenario.

## 4. Open Questions

- Which start locations are supported?
- How is a feasible journey defined?
- What information is mandatory in the results?
- How are missing data and outdated snapshots handled?
