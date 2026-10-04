# Navora Marine MVP Implementation

## Goal

Help a fleet operator determine whether an arriving vessel appears ready to reach its planned berth, identify blockers and uncertainty, and compare arrival options. Navora is a decision-support prototype. It does not grant port clearance, reserve services, or issue navigational instructions. The vessel's master, pilot, port control, terminal, and authorities retain their responsibilities.

## MVP User and Workflow

Primary user: fleet/voyage operator coordinating through the ship's agent.

1. Find a vessel with an arrival risk on the operations overview.
2. Review vessel, berth, service, channel, clearance, and environmental readiness.
3. Compare maintain speed, slow down, anchor, and wait offshore.
4. Save a proposed plan and prepare a confirmation request for the agent/port.
5. Record the operator's decision and later updates in the history.

For a short build, support one port, one vessel, one berth, and one coherent scenario. Use actual public measurements where available and clearly labeled sample data for private operational information.

## Feature Inputs

### Vessel

- Name and IMO number
- Length overall, beam, arrival draft, and cargo type
- Current ETA and speed
- **Current position or remaining distance/time to the pilot boarding point**
- Source, last-updated time, and confidence for dynamic values

ETA and speed alone cannot calculate the new ETA after a speed change. Position or remaining route distance is required. Track ETA to the pilot boarding point separately from ETA at berth.

### Destination

- Port, terminal, and assigned berth
- Assignment state: proposed, confirmed, or unconfirmed
- Pilot boarding point
- Designated anchorage/offshore holding location
- **Requested or confirmed time at the pilot boarding point**

### Berth

- Berth identifier, usable length and depth, cargo capability
- Maximum beam/draft restrictions when published
- Current occupant and expected availability (time or range)
- Availability source, confidence, and update time

### Arrival services and access

- Pilot time/availability
- Tug requirement and availability (count/type/capacity, not only yes/no)
- Mooring crew/linesmen availability
- Channel slot
- **Fairway/channel operational status, separate from the slot**
- Port clearance: cleared, pending, blocked, or unknown

### Environment

- Tide/water level and usable tide window
- Weather, wind, current, and visibility
- Source, timestamp, units, and location for each value
- **Port-required under-keel clearance (UKC) or safety margin**

Depth and tide must use compatible units and vertical datums. Do not treat a predicted tide window as permission to enter or proof of safe depth.

### Arrival options

- Maintain speed
- Slow down
- Anchor at a designated anchorage
- Wait offshore at a designated holding area

Each option shows estimated pilot boarding time, berth arrival, waiting time/location, blockers, and assumptions. Show fuel impact only when vessel fuel-use data supports it; otherwise show “Fuel estimate unavailable.”

## Pages and Buttons

### Operations Overview

Show vessels needing attention, destination, ETA, readiness summary, limiting blocker, and latest update.

- **View vessel**: open vessel details.
- **Review arrival**: open its arrival planner.
- **Filter**: filter by port, readiness, or arrival time.
- **Acknowledge**: record alert review; does not clear the blocker.

### Vessel Detail

Show particulars, AIS position/speed if connected, ETA to pilot point, destination, readiness, and event history.

- **Edit vessel**: update static particulars.
- **Update ETA**: record ETA, source, and time.
- **Plan arrival**: open comparison.
- **View port**: open port/terminal/berth detail.
- **Add update**: record a service or port status update with its source.
- **View changes**: show who changed what and when.

### Port and Berth Detail

Show published berth limits, assignment, occupant, expected release, channel/fairway status, environmental stations, and service readiness.

- **View berth schedule**: show planned occupancy and availability ranges.
- **Request confirmation**: prepare an agent/port request for selected unknown or stale fields.
- **Update status**: edit manually supplied/sample status while preserving source and timestamp.

### Arrival Planner

Compare the four options in a table or option rows. Each shows timing, wait location, fuel estimate if supported, readiness, blockers, and assumptions.

- **Compare options**: recalculate scenarios from current inputs.
- **Select option**: save choice as a draft.
- **Request confirmation**: create a message draft with proposed arrival time and unresolved requirements.
- **Accept plan**: record the operator's decision and assumptions in history. This is internal acceptance, not port approval.

Without messaging integration, show “Draft, not sent.” Never display a request or booking as completed unless an external system received it.

Reports and Settings are optional for the first 10 hours. Do not build them at the expense of the complete arrival workflow.

## Readiness and Calculation Rules

Use four statuses:

- **Ready**: confirmed available at the proposed time.
- **At risk**: expected to be available, but uncertain, stale, or close to the needed time.
- **Blocked**: a known conflict prevents this plan.
- **Unknown**: missing or unconfirmed information.

Every status has a reason, source, update time, and confidence. Unknown must never silently become Ready.

Deterministic calculation sequence:

1. Check vessel/berth fit: length, beam, arrival draft, cargo, and published restrictions.
2. Check water depth using applicable depth, tide/water level, compatible datum, and UKC margin.
3. Estimate time to pilot boarding point from position/distance, route duration, and proposed speed.
4. Check berth, fairway/channel, pilot, tug, mooring crew, clearance, and environmental constraints at that time.
5. Find the earliest time required dependencies are expected to be ready; preserve time ranges and uncertainty.
6. Compare options and explain the limiting dependency. Highlight stale or unknown inputs.
7. Save the assumptions/input snapshot for any accepted plan.

If a required input is unknown, show the limitation and request confirmation. Never claim “safe to enter.” Demo recommendations are planning estimates, not navigational advice or clearance.

## Data Sources

No single public source provides all fields.

- **U.S. water levels, tide predictions, currents, wind, visibility:** [NOAA CO-OPS Data API](https://api.tidesandcurrents.noaa.gov/api/dev) and [station metadata](https://api.tidesandcurrents.noaa.gov/mdapi/prod/). Check that a station provides each needed measurement.
- **U.S. marine forecasts and alerts:** [National Weather Service API](https://www.weather.gov/documentation/services-web-api); coverage and marine forecast fields vary by location.
- **U.S. charted navigation features/depths:** free [NOAA ENC data](https://www.nauticalcharts.noaa.gov/charts/noaa-enc.html). It is navigation context, not live berth status or guaranteed current berth depth.
- **Historical U.S. vessel traffic:** [MarineCadastre AIS archive](https://marinecadastre.gov/accessais/), useful for replay rather than live position.
- **Live vessel position/speed:** AIS Stream or a commercial provider such as [MarineTraffic AIS API](https://servicedocs.marinetraffic.com/), subject to access, coverage, rate limits, and terms.
- **Vessel particulars:** operator-provided data or a vessel data provider; AIS static values can be missing or stale.
- **Port/terminal/berth reference data:** port authority and terminal port guides, facility pages, and published restrictions.
- **Actual berth assignment/availability, previous vessel departure, pilot/tug/mooring schedules, channel allocation, and clearance:** usually require port, terminal, agent, service provider, or authorized port-system data. AIS can suggest a vessel is alongside but cannot confirm its booking or departure. IMO's [port-call optimization guidance](https://greenvoyage2050.imo.org/port-call-optimization-guide-launched-at-imo/) describes operational data exchange among port stakeholders.

Hackathon default: use NOAA readings and real or replayed AIS where feasible; seed private operational fields with an explicitly labeled “Demo port feed” or “Sample agent confirmation.” Show source and last-updated time on every value.

## Data Model

- `Vessel`: IMO number, name, dimensions, arrival draft, cargo type.
- `Voyage`: vessel, destination, terminal/berth, ETA/PBP, position/speed, requested arrival time.
- `Port`, `Terminal`, `Berth`: location, limits, capability, restrictions.
- `RequirementStatus`: requirement, status, planned time, source, updated time, confidence, reason.
- `EnvironmentReading`: location, type, value/unit, datum when relevant, observation/forecast time, source.
- `ArrivalPlan`: option, speed/holding location, estimated milestones, blockers, assumptions, fuel estimate if supported, selection state.
- `PlanEvent`: actor, action, changed value, timestamp, source/note.

Store timestamps in UTC and render in port local time. Store units and datum explicitly. Keep source references and input snapshots for explainability.

## Technical Approach

Use a conventional web app, small API/backend, and relational database. Keep readiness rules deterministic. Store time-stamped events for freshness and history. Put external providers behind adapters so sample and live data share one internal format. Keep private API keys server-side.

TigerData can store/query time-series data; it does not supply AIS or port feeds. Gemini can summarize structured blockers or draft a message, but must not invent missing statuses or determine readiness. The app must work in sample-data mode without provider credentials.

Possible server-side environment variables: `DATABASE_URL`, `AIS_PROVIDER_API_KEY`, `NOAA_USER_AGENT`, optional `GEMINI_API_KEY`.

## Ten-Hour Build Order

1. **1 hour:** choose a port and define one internally consistent scenario.
2. **2 hours:** build overview, vessel detail, and shared readiness components.
3. **2 hours:** build port/berth detail and readiness calculations.
4. **2 hours:** build the four-option arrival planner.
5. **1 hour:** add draft confirmation, operator acceptance, and history.
6. **2 hours:** connect one available live source, label sample fields, polish, deploy, and prepare the demo.

If provider setup takes too long, keep the working sample scenario and label it honestly; do not present simulated data as live.

## Completion Criteria

- Overview shows at least one vessel needing attention.
- Vessel/berth compatibility is reviewable.
- Each readiness item shows status, reason, source, and update time.
- All four options are compared and the limiting dependency is explained.
- Unknown/stale inputs remain visible and affect the result.
- Operator can select/accept a draft plan and see it in history.
- Confirmation clearly indicates whether it was sent or remains a draft.
- Live and sample data are visibly distinguished.

## Later Scope

After the marine flow works end to end, consider real port/terminal integrations, external messaging, multi-port voyage planning, stronger fuel estimates, and aviation/road modules. Keep those domain rules in their own modules.
