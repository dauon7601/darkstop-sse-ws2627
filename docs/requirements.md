
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

## 5. Proposed Acceptance Test Cases

The following test cases are planned and have not
yet been executed.

### TC-06: Valid Travel Window
Related Requirement: FR-01

Input:
- Starting location: Frankfurt Hbf
- Departure: November 28, 2026, 19:00
- Return deadline: November 29, 2026, 01:30

Expected Result:
The system accepts the input as a valid travel window.

### TC-07: Missing Starting Location
Related Requirement: FR-01

Input:
- Starting location: Not specified
- Departure: November 28, 2026, 19:00
- Return deadline: November 29, 2026, 01:30

Expected Result:
The system rejects the request and displays
a clear validation message.

### TC-08: Invalid Travel Window
Related Requirement: FR-01

Input:
- Departure: November 28, 2026, 19:00
- Return deadline: November 28, 2026, 18:00

Expected Result:
The system rejects the invalid travel window.

### TC-09: Reproducible Results
Related Requirement: NFR-01

Input:
Two identical requests using the same data snapshot.

Expected Result:
Both requests return identical destinations,
journey details, scores, and ranking order.

### TC-10: Display of Recommendation Details
Related Requirement: FR-06

Input:
A predefined test destination with a feasible
outbound and return journey.

Expected Result:
The interface displays the destination name,
sky darkness, cloud cover, travel times,
walking information, outbound and return trips,
and the snapshot timestamp.

### TC-11: Offline Prototype Startup
Related Requirement: NFR-03

Precondition:
The required Docker environment is installed.

Steps:
1. Disable access to live third-party services.
2. Run `docker compose up`.
3. Execute the predefined sample scenario.

Expected Result:
The application starts successfully and processes
the sample scenario without external API calls
or API keys.

