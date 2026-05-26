# Pyra

Pyra is an AI-assisted wildfire command intelligence platform built around a live operational map, incident-centered resource coordination, and dispatcher-controlled tactical workflows.

It brings wildfire incidents, responding units, alerts, weather, air quality, terrain, routing, spread risk, fire perimeters, and operational recommendations into one command surface. The goal is to help command staff understand what is happening, compare response options, and coordinate resources with more context than a static dashboard can provide.

## Links

- Pyra: https://pyra-ai.vercel.app/

## Why Pyra

Wildfire response is a fast-moving coordination problem. Incident commanders and dispatchers have to reason across changing fire behavior, limited resources, route conditions, atmospheric risk, unit availability, and competing incidents. Those signals often live in separate systems, which makes it harder to maintain a shared operational picture.

Pyra is designed to collapse those signals into a single tactical workspace. The map is the center of the experience, while intelligence panels, unit recommendations, alert triage, briefings, and incident detail views stay close to the action. Instead of treating data feeds as isolated widgets, Pyra normalizes them into incident state, operational overlays, routing context, and recommendation logic.

The result is a command tool focused on practical questions:

- Which incidents need attention first?
- Which units are best positioned for this fire?
- What route posture and safety context matters?
- What does weather, AQI, terrain, and spread risk imply?
- What resources are already committed?
- What briefing or handoff context should responders see?

## Product Description

Pyra combines four major layers into one command platform:

1. Live incident state: fires, unit positions, alerts, containment, weather, AQI, terrain, fire perimeters, and operational activity.
2. Operational intelligence: fire-behavior estimates, composite risk scoring, spread-risk modeling, route safety scoring, and resource recommendation logic.
3. AI assistance: dispatch advice, tactical summaries, alert triage, loadout suggestions, SITREP chat, operational briefings, handoff briefings, PDF reports, and post-incident review.
4. Command UX: a map-first interface with floating panels for incident detail, engaged units, unified activity, unit status, and cross-incident prioritization.

Pyra ships as both a web application and an Electron desktop application. The desktop shell packages the React frontend and FastAPI backend into a native app experience, with local backend process management and settings integration through Electron IPC.

## Interface Model

The frontend is a map-first React and Leaflet application. The map remains the main workspace, while panels act as tactical overlays instead of separate pages.

Core interface surfaces include:

- Top bar: system metrics, health state, overlay toggles, command view access, and settings.
- Left sidebar: engaged units for the selected incident, filtered to active operational states.
- Right panel: unified activity feed combining alerts and timeline events, plus a system-wide unit roster.
- Incident detail panel: the primary tactical workspace for selected incident analysis and action.
- Command view: a multi-incident prioritization surface for comparing risk and allocation needs across the active incident set.
- Settings panel: user-level configuration and Electron-specific API key management.

The interaction model keeps tactical actions close to the map. Selecting a fire opens incident context, selecting a unit focuses route and status details, and overlays can be toggled to reveal evacuation zones, fire growth, perimeters, heatmaps, weather, water sources, and satellite context.

## Core Operational Surfaces

### Live Map

The live map renders severity-coded fire markers, smoothed unit motion, callsign labels, route previews, active operational routes, fire growth, risk overlays, evacuation zones, water sources, satellite imagery, and weather context.

Camera movement uses eased map transitions for selected fires, units, routes, and incident bounds. The map supports follow-style unit tracking and keeps route geometry visible as dispatch decisions are evaluated.

### Incident Detail

The incident detail panel provides the deepest operational workflow in Pyra. It brings together recommendation profiles, confidence levels, situation summaries, fire intelligence metrics, required unit types, committed units, candidate unit ranking, dispatch advice, loadout configuration, SITREP chat, briefing generation, report export, close-out workflow, and post-incident review.

### Activity System

The activity feed merges alerts and incident timeline events into a single operational stream. Alerts include severity, incident association, acknowledgement state, and AI triage. Timeline events capture incident lifecycle updates and detection activity. The panel can collapse into a narrow rail to preserve map space while keeping new activity visible.

### Command View

Command view ranks incidents using composite risk logic and presents a broader allocation picture across the system. It is built for comparing active fires, understanding relative priority, and deciding where scarce resources should move next.

## Intelligence and Decision Logic

Pyra uses deterministic scoring engines together with AI-assisted interpretation.

### Deterministic Engines

- `recommendation_engine.py` produces incident recommendations, loadout profiles, unit type demand, confidence, and tactical notes from incident attributes and route context.
- `unit_selection.py` converts recommendation demand into actual candidate units by subtracting committed resources, ranking available units, and reporting shortfalls.
- `route_safety.py` and `routing.py` build route geometry, assign route safety state, and normalize route reasoning.
- `fire_behavior.py`, `composite_risk.py`, and `spread_risk.py` estimate fire behavior, composite incident risk, and directional spread posture from weather, terrain, AQI, and incident state.

### Simulation Engine

The backend continuously evolves operational state so the UI behaves like a moving command picture rather than a manually refreshed dashboard.

Simulation mechanics include:

- Per-incident containment progression based on on-scene unit contribution.
- Weather variation around base wind and humidity readings.
- Weather-driven alert generation for dangerous wind and humidity conditions.
- Alert pruning to keep the operational feed usable.
- Unit movement, route updates, and incident state refreshes.

### AI-Assisted Flows

Anthropic-backed endpoints are used for reasoning-heavy workflows:

- Dispatch advice for selected unit sets.
- Per-unit loadout recommendations from structured equipment data.
- Alert triage summaries.
- Operational incident briefings.
- Handoff briefings.
- SITREP chat.
- Post-incident AAR review.

Caching is used for repeated AI workflows such as loadout generation and alert triage, keeping the interface responsive while preserving context-aware output.

## Data Sources and Enrichment

Pyra fuses internal operational state with external data feeds:

- Open-Meteo for weather enrichment.
- Open-Meteo AQ for air quality enrichment.
- Open-Elevation for terrain data.
- NASA FIRMS for hotspot ingestion and satellite-aware incident updates.
- Overpass and OpenStreetMap for water sources and road-derived context.
- NIFC ArcGIS for fire perimeters.
- OSRM for road routing.

These feeds are normalized into incident attributes, map overlays, route context, recommendation scoring, alert generation, and briefing material.

## Technical Breakdown

### Architecture

Pyra is organized as a full-stack application with three primary layers:

- `backend/`: FastAPI application for APIs, persistence, scheduling, simulation, enrichment, AI workflows, audit logging, and operational services.
- `frontend/`: React and Vite application for the map-first command interface.
- `electron/`: Native desktop wrapper that starts the backend, loads the frontend, and exposes secure IPC for local settings and backend restart control.

The backend uses PostgreSQL for persistent operational state, SQLAlchemy for data modeling, Alembic for migrations, APScheduler for recurring jobs, and SlowAPI for rate limiting sensitive endpoints.

### Backend API Domains

- `/api/auth`: authentication and current-user identity.
- `/api/incidents`: incident state, close-out checklist, and closure actions.
- `/api/units`: unit inventory and location updates.
- `/api/alerts`: alert lifecycle, acknowledgement, clearing, and statistics.
- `/api/dispatch`: dispatch approval and alert dispatch.
- `/api/recommendations`: dispatch intelligence and feedback capture.
- `/api/dispatch-advice`: AI assessment of proposed dispatch selections.
- `/api/dispatch/loadout`: AI loadout recommendation and fallback handling.
- `/api/intelligence`: fire behavior, spread risk, recommendation summaries, alert recommendations, and intelligence aggregation.
- `/api/routes`: route safety and route-related metadata.
- `/api/water-sources`: water source discovery and operational support data.
- `/api/multi-incident`: cross-incident prioritization.
- `/api/briefing`: operational briefing and handoff generation.
- `/api/chat`: SITREP chat stream.
- `/api/review`: post-incident review stream.
- `/api/report`: PDF incident report export.
- `/api/audit`: audit log viewing, checksum verification, and CSV export.
- `/api/ingestion`: manual and scheduled ingestion status.
- `/health`: backend health and component readiness.

### Background Jobs

Scheduled backend services keep operational state current:

- simulation updates
- route generation and refresh
- weather enrichment
- AQI enrichment
- FIRMS ingestion
- terrain enrichment
- road and access enrichment

### Frontend Components

Primary frontend components include:

- `App.jsx`: top-level orchestration, polling, auth gate, panel composition, keyboard shortcuts, and layout geometry.
- `IncidentMap.jsx`: map rendering, markers, camera control, route overlays, follow mode, and map legend.
- `LeftSidebar.jsx`: incident-specific engaged-unit rail.
- `RightPanel.jsx`: unified activity feed and system unit roster.
- `IncidentDetailPanel.jsx`: tactical incident workspace.
- `DispatchRecommendations.jsx`: candidate ranking and dispatch selection flow.
- `LoadoutConfigurator.jsx`: per-unit loadout configuration.
- `TopBar.jsx`: metrics, controls, and overlay toggles.
- `SettingsPanel.jsx`: user settings and Electron API key controls.
- `AuditLogPanel.jsx`: audit log viewing and integrity verification.
- `PostIncidentReview.jsx`: streaming AAR surface.
- `SitrepChat.jsx`: interactive incident Q&A.
- `WeatherPanel.jsx`: atmospheric conditions panel.
- `Toast.jsx`: operational feedback notifications.

Map overlay components include evacuation zones, fire growth, fire perimeters, composite risk heatmaps, spread risk, and water sources.

### Roles and Permissions

Pyra uses role-based access control:

- `viewer`: read access to operational data.
- `dispatcher`: dispatch operations, alert acknowledgement, and briefing-related actions.
- `commander`: full operational access, including close-out, review, and audit verification workflows.

### Runtime Stack

Desktop:

- Electron 28
- electron-builder 24

Backend:

- FastAPI 0.115.0
- SQLAlchemy 2.0.36
- Pydantic 2.9.2
- APScheduler 3.10.4
- Anthropic SDK 0.40.0
- Alembic 1.13.3
- SlowAPI 0.1.9
- PostgreSQL 16

Frontend:

- React 19.2.4
- React DOM 19.2.4
- React-Leaflet 5.0.0
- Leaflet 1.9.4
- Vite 8.0.0
- Tailwind CSS 4.2.1

## Repository Layout

```text
Pyra/
|-- electron/
|   |-- main.js
|   `-- preload.js
|-- backend/
|   |-- alembic/
|   |-- app/
|   |   |-- api/
|   |   |-- core/
|   |   |-- ext/
|   |   |-- intelligence/
|   |   |-- models/
|   |   |-- schemas/
|   |   |-- scripts/
|   |   |-- services/
|   |   `-- utils/
|   `-- tests/
|-- frontend/
|   `-- src/
|       |-- api/
|       |-- components/
|       |-- context/
|       |-- services/
|       `-- utils/
|-- package.json
|-- docker-compose.yml
`-- README.md
```
