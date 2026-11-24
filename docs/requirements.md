
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

### Revised Goal

By the final demo, DarkStop answers one question for a person
without a car: which dark-sky spots in the RMV region can be reached
by public transport on a chosen night, under the clearest sky, with
a planned return before a chosen deadline. Every answer is computed
from one versioned data snapshot, so it is reproducible and testable.

Changes since the revised proposal (November 15, 2026):
- "Feasible" is defined precisely (FR-04): outbound journey, return
  journey before the deadline, and at least 60 minutes on site
  during astronomical darkness.
- Time on site is part of feasibility, not only travel.
- The cloud forecast is captured in the snapshot. No external API
  is called at runtime (NFR-01, NFR-03).
- Requirement IDs follow the proposal numbering. New requirements
  are appended (FR-09 onwards, NFR-05 onwards).

Definition of success: all test cases for Must requirements pass
in CI on `main` against the frozen fixtures, and the prototype
starts with `docker compose up` without network access.

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
- Departure and return deadline are full date-times with time zone
  (ISO 8601, for example `2026-11-28T19:00+01:00`). The frontend may
  offer a date plus two times and converts them before sending.
- Supported starting locations are the major Frankfurt stations
  contained in the snapshot.
- A window shorter than 2 hours or longer than 12 hours is rejected
  with the code `WINDOW_INVALID`.
- A date outside the snapshot coverage is rejected with the code
  `DATE_OUT_OF_RANGE`.

### FR-02: Candidate Locations
Priority: Must

Requirement:
The system shall evaluate only candidate locations from a fixed,
versioned spot catalogue.

Acceptance Criteria:
- Each catalogue entry has an ID, name, coordinates, sky brightness
  value and at least one access stop within 3 km.
- Every spot has public access.
- The catalogue is reviewed by both team members before a snapshot
  is built.

### FR-03: Outbound and Return Journeys
Priority: Must

Requirement:
The system shall determine outbound and return journeys from the
timetable snapshot.

Acceptance Criteria:
- The outbound journey is the earliest journey (transit + walk)
  departing at or after the earliest departure.
- The return journey is the latest journey arriving at the
  starting location by the return deadline.
- Trips running after midnight are found, including trips stored
  with times after 24:00 on the previous GTFS service day.

### FR-04: Feasibility and Exclusion
Priority: Must

Requirement:
The system shall exclude destinations without a feasible return
before the deadline.

Acceptance Criteria:
- A destination is feasible only if an outbound and a return
  journey exist and the time on site overlaps astronomical
  darkness (sun below -18°) by at least 60 minutes.
- Infeasible destinations are excluded with a reason code, for
  example `NO_RETURN_BEFORE_DEADLINE` or `TIME_ON_SITE_TOO_SHORT`.
- Destinations with missing data get the status `UNKNOWN` and are
  never shown as feasible.

Feasibility parameters (part of the snapshot configuration, so a
change produces a new snapshot ID):

| Parameter | Default | Rationale |
| --- | --- | --- |
| Minimum time on site | 60 min | Dark adaptation alone takes 20 to 30 min |
| Darkness threshold | Sun below -18° | Astronomical darkness, see open question Q1 |
| Maximum walk stop to spot | 3.0 km per direction | Darker ground is usually 1 to 3 km from the stop |
| Walking speed | 4.5 km/h | Slower than by day because of darkness and terrain |
| Minimum transfer time | OpenTripPlanner default | No custom transfer model |

### FR-05: Ranking
Priority: Must

Requirement:
The system shall rank feasible destinations by sky darkness,
cloud cover and walking distance.

Acceptance Criteria:
- The score combines sky brightness, mean cloud cover during the
  time on site and walking distance with documented weights.
- Ties are broken by longer time on site, then by spot ID.
- The weights are recorded in an ADR in `docs/adr/`.

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
- Sky quality is shown in mag/arcsec² and as an approximate Bortle
  class.
- The score breakdown (darkness, clouds, walk) is shown per
  destination.
- The snapshot ID and the build dates of timetable and forecast
  are shown on every result page.

### FR-07: Lit Share and Fallback Return
Priority: Should

Requirement:
The system shall show the lit share of the walk from stop to spot
and a later fallback return journey.

Acceptance Criteria:
- Lit share is computed from OSM `lit` tags; untagged segments
  count as unlit.
- The fallback return is the next later return that still arrives
  before 06:00, if one exists.

### FR-08: Map and List View
Priority: Should

Requirement:
The system shall show results on a map and as a list.

Acceptance Criteria:
- Map and list show the same destinations in the same order.

### FR-09: Moon Information
Priority: Could

Requirement:
The system shall show moon phase and moon altitude during the
time on site.

### FR-10: Snapshot Rebuild Job
Priority: Could

Requirement:
A scheduled job shall rebuild the snapshot with a fresh forecast,
producing a new snapshot ID.

### Out of Scope (Won't, this term)

- Live delays and cancellations (GTFS-RT)
- Aurora alerts and push notifications
- User accounts, saved or shared trips
- Native mobile apps
- Free address input and geocoding

## 3. Non-Functional Requirements

### NFR-01: Reproducibility
Priority: Must

Requirement:
Identical inputs and the same data snapshot shall produce
identical results.

Acceptance Criterion:
Repeated requests with identical inputs and snapshot versions
produce identical results.

### NFR-02: Fail-Safe Data Handling
Priority: Must

Requirement:
Missing, unparseable or stale data shall never count as proof of
a feasible journey.

Acceptance Criterion:
Test cases with missing journey data produce the status `UNKNOWN`
and no feasible result (TC-03).

### NFR-03: Reproducible Execution
Priority: Must

Requirement:
The prototype shall run with bundled sample data using
Docker Compose, without requiring live third-party services.

Acceptance Criterion:
A clean test environment can start the prototype using
`docker compose up` and execute the agreed sample scenario.

Target: a machine with Docker and 16 GB RAM, no API keys, no network
after images and snapshot are pulled; UI ready in under 10 minutes.

### NFR-04: Performance
Priority: Should

Requirement:
A ranked result shall be returned in under 5 seconds on the demo
machine.

Acceptance Criterion:
p95 response time under 5 s over 20 sequential requests with 30
candidate spots on a warm system (TC-16).

### NFR-05: Testability
Priority: Must

Requirement:
The clock shall be injectable and all routing tests shall run
against synthetic GTFS fixtures.

Acceptance Criterion:
The full acceptance test suite runs in CI in under 10 minutes.

### NFR-06: API Contract Stability
Priority: Must

Requirement:
`docs/openapi.yaml` shall be the single source of truth for the API
between backend and frontend.

Acceptance Criterion:
Backend contract tests fail on any mismatch with the specification.

### NFR-07: Licensing
Priority: Must

Requirement:
Only derived per-spot sky values shall be bundled; all data sources
shall be credited in the UI.

Acceptance Criterion:
The UI footer credits DELFI e.V. (CC BY 4.0), OpenStreetMap
contributors (ODbL) and Open-Meteo (CC BY 4.0); the raw light
pollution atlas is not in the repository or the snapshot.

### NFR-08: Usability
Priority: Should

Requirement:
A first-time user shall find a feasible spot and its return journey
without help.

Acceptance Criterion:
3 of 3 first-time test users succeed in under 2 minutes.

### NFR-09: Night Use
Priority: Should

Requirement:
The UI shall be dark by default and readable at night.

Acceptance Criterion:
Dark theme by default; text contrast meets WCAG 2.1 AA.

### NFR-10: Code Quality
Priority: Should

Requirement:
The feasibility and ranking logic shall be covered by unit tests.

Acceptance Criterion:
At least 80 % line coverage on the feasibility and ranking packages
(JaCoCo report in CI).

## 4. Open Questions

- Which start locations are supported?
  Resolved: the major Frankfurt stations contained in the snapshot
  (FR-01).
- How is a feasible journey defined?
  Resolved: outbound journey, return before the deadline and at
  least 60 minutes on site during astronomical darkness (FR-04).
- What information is mandatory in the results?
  Resolved: see the acceptance criteria of FR-06.
- How are missing data and outdated snapshots handled?
  Resolved: missing data gives `UNKNOWN`, never feasible (NFR-02);
  every result names its snapshot (FR-06).

Open for class discussion:

- Q1: Frankfurt has no astronomical night from about May 31 to
  July 11. Fall back to nautical darkness (-12°) and flag it, or
  limit the product to the other months?
- Q2: Is 60 minutes the right minimum time on site?
- Q3: How can the ranking be validated without ground truth?
  Proposal: known dark sites must rank high, spots near Frankfurt
  low.
- Q4: Is a frozen forecast acceptable for a reproducible prototype?
- Q5: Should the walking limit drop from 3 km to 1.5 km for the
  primary user?

## 5. Proposed Acceptance Test Cases

The following test cases are planned and have not
yet been executed.

All test cases use the synthetic fixture snapshot `fx-01`, covering
November 28 to 30, 2026, starting location Frankfurt Hbf, departure
November 28, 2026, 19:00 and return deadline November 29, 2026,
01:30, unless stated otherwise. Astronomical darkness starts at
18:25 on that night.

The prototype is accepted when all test cases for Must requirements
pass: TC-01 to TC-15 as automated tests in CI, TC-11 on the demo
machine. TC-16 to TC-18 cover Should requirements.

### TC-01: Feasible Destination
Related Requirement: FR-03, FR-04

Input:
- Spot A: outbound departs 19:05, arrives 20:10
- Return departs 23:40, arrives Frankfurt Hbf 00:30

Expected Result:
Spot A is shown as feasible with both journeys and 3 h 30 min
on site.

### TC-02: No Return Before Deadline
Related Requirement: FR-04

Input:
- Spot B: the last return arrives 01:45

Expected Result:
Spot B is not shown as feasible; reason `NO_RETURN_BEFORE_DEADLINE`.

### TC-03: Missing Return Data
Related Requirement: FR-04, NFR-02

Input:
- Spot C: return journey data is missing

Expected Result:
Spot C has the status `UNKNOWN`, is not ranked and is never shown
as feasible.

### TC-04: Darker Spot Ranks Higher
Related Requirement: FR-05

Input:
- Spots D and E identical except sky brightness 21.4 vs 20.6
  mag/arcsec²

Expected Result:
Spot D ranks above spot E.

### TC-05: Return After Midnight
Related Requirement: FR-03

Input:
- Spot F: the only return departs 00:40, stored as 24:40 on the
  November 28 service day

Expected Result:
The return is found and spot F is feasible.

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

### TC-12: Date Outside Snapshot
Related Requirement: FR-01

Input:
- Departure: December 12, 2026, 19:00
- Return deadline: December 13, 2026, 01:30

Expected Result:
The request is rejected with `DATE_OUT_OF_RANGE`; no results.

### TC-13: Time on Site Too Short
Related Requirement: FR-04

Input:
- Spot G: outbound arrives 22:10, the last return departs 22:50

Expected Result:
Spot G is not feasible; reason `TIME_ON_SITE_TOO_SHORT` (40 min is
under the 60 min minimum).

### TC-14: Clearer Spot Ranks Higher
Related Requirement: FR-05

Input:
- Spots H and J identical except mean cloud cover 10 % vs 70 %

Expected Result:
Spot H ranks above spot J.

### TC-15: Tie-Break
Related Requirement: FR-05

Input:
- Spots K and L with equal score; K has 30 min more on site

Expected Result:
Spot K ranks above spot L. With equal time on site, the lower spot
ID ranks first.

### TC-16: Response Time
Related Requirement: NFR-04

Input:
20 sequential requests on a warm system with the real snapshot and
30 candidate spots.

Expected Result:
p95 response time is under 5 seconds.

### TC-17: Map and List Consistency
Related Requirement: FR-08

Input:
Any result set.

Expected Result:
Map and list show the same destinations in the same order.

### TC-18: Lit Share
Related Requirement: FR-07

Input:
- A 2.0 km walk: 0.5 km tagged `lit=yes`, 1.5 km untagged

Expected Result:
The lit share is shown as 25 %.

## 6. Scope and Users

In scope this term:
- Major Frankfurt stations as starting locations
- Curated catalogue of about 30 spots in the RMV region
- Planned timetable (GTFS static)
- Cloud forecast captured at snapshot build time
- Web app in desktop and mobile browsers, anonymous use

Primary user: student without a car, with a Deutschlandticket, who
wants to see the Milky Way once. Needs a trustworthy return
connection more than the darkest possible sky.

Secondary user: hobby astrophotographer. Needs several hours on
site and hourly cloud cover; accepts a longer walk for a darker sky.

## 7. Data Sources

| Source | Used for | Licence and constraint |
| --- | --- | --- |
| [DELFI GTFS Germany](https://www.transit.land/feeds/f-delfi~de), cut to the RMV region | Planned timetable | CC BY 4.0, attribution to DELFI e.V.; planned times only |
| OpenStreetMap, Hessen extract | Walking network, `lit` tags | ODbL; `lit` tags are patchy in rural areas |
| [World Atlas of Artificial Night Sky Brightness](https://b2find.eudat.eu/dataset/df8cd562-df6a-51bf-ad64-7421cf8c8623) (Falchi et al. 2016) | Sky brightness per spot | [Paper CC BY-NC](https://repository.library.noaa.gov/view/noaa/15730); only derived per-spot values bundled; artificial radiance in mcd/m² converted to mag/arcsec² |
| [Open-Meteo](https://open-meteo.com/en/licence), DWD ICON | Hourly cloud cover | Data CC BY 4.0, [free API non-commercial only](https://open-meteo.com/en/terms); link required where shown |
| Sun and moon ephemeris | Darkness window, moon | Computed locally |
| Spot catalogue (team) | Candidate spots | CSV in the repository, reviewed by both members |

`data/manifest.json` records every source URL, retrieval date and
SHA-256 plus the feasibility parameters. One build produces one
snapshot ID.

## 8. Team Organisation

| Member | Owns | Shares |
| --- | --- | --- |
| Hafeezullah Haidari | Spring Boot API, feasibility and ranking, OpenTripPlanner integration, OpenAPI contract | Data preparation, spot catalogue, requirements, reviews, tests, docs |
| Da-Uon Kim | PostgreSQL/PostGIS schema, data validation, Next.js frontend with MapLibre | Data preparation, spot catalogue, requirements, reviews, tests, docs |

| Format | When | Purpose |
| --- | --- | --- |
| Weekly sync, 60 min | Every Tuesday | Review board, plan the week, settle open questions |
| Async check-in | Every Friday, team chat | Done / next / blocked |
| Sprint review, 20 min | Every second Tuesday | Demo merged work against the test cases |
| Integration pairing | Before each milestone | Connect API and frontend, run the acceptance suite |

Decision rules:
1. Inside a component, its owner decides and documents it in the
   pull request.
2. API contract, schema, scoring weights and scope need both members
   and are recorded as an ADR in `docs/adr/`.
3. Without agreement after one weekly sync, the option with less
   scope wins; scope questions go to the instructor.

Definition of Done:
- Merged to `main` through a pull request approved by the other
  member.
- CI green: build, unit tests, acceptance tests, linters.
- The linked issue names the requirement ID; its test case passes.
- OpenAPI specification, schema migration and README updated where
  affected.

## 9. Repository Workflow

- `main` is protected; no direct pushes.
- One short-lived branch per issue: `feat/FR-05-ranking`, `fix/...`,
  `docs/...`.
- Conventional Commits (`feat:`, `fix:`, `test:`, `docs:`, `chore:`)
  with the requirement ID in the body.
- Pull requests need one approval and green CI; squash merge.
- Raw GTFS, OSM and atlas files are never committed; the built
  snapshot ships as a GitHub Release asset.
- Tag `v0.x` at each milestone.

Planned additions to the repository: `tests/fixtures/`,
`docs/openapi.yaml`, `docs/adr/`, `otp/`, `.github/` (CI, templates,
CODEOWNERS), `docker-compose.yml`.

## 10. Milestones

| Date | Milestone |
| --- | --- |
| Nov 26, 2026 | Draft requirements and slides submitted |
| Nov 28, 2026 | Class presentation |
| Nov 29 to Dec 8 | Sprint 0: repository, CI, API contract v0, schema v0, fixture `fx-01` |
| Dec 9 to Dec 22 | Sprint 1: vertical slice, TC-01 to TC-03 and TC-06 to TC-08 pass |
| Jan 5 to Jan 19, 2027 | Sprint 2: all Must requirements, all Must test cases pass |
| Jan 20 to Feb 2, 2027 | Sprint 3: Should requirements, usability test |

The final demo date follows the course calendar.

## 11. Risks

| Risk | Mitigation | Owner |
| --- | --- | --- |
| OpenTripPlanner graph needs more RAM than the demo machine has | Cut GTFS and OSM to a bounding box; measure in Sprint 0 | Hafeez |
| Night connections missing or wrong in GTFS | Journeys labelled "planned"; spot-check 5 returns against the RMV planner per snapshot | Shared |
| Midnight and service-day handling | TC-05 and TC-08 pass before any ranking work | Hafeez |
| Atlas licence restricts redistribution | Bundle only derived per-spot values (NFR-07) | Da-Uon |
| One member unavailable | Must requirements only until all Must test cases pass | Shared |