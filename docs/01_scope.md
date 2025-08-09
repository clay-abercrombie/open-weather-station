# 01 – Weather Station Scope

> Status: **Locked v0.1**  
> Owner: Clay Abercrombie  
> Repo: `open-weather-station`  
> Audience: contributors, reviewers, and future maintainers

---

## 1) Mission Statement
Continuously log local weather and surface it on a local (LAN) dashboard for farm operations, with the option to export/sync to cloud later.

---

## 2) Context & Environment
- **Location/Climate:** Gulf Coast (Houston area). High heat, direct sun, heavy rain/winds, salt-laden humidity, occasional hail, freezing events, dust/insects/birds.
- **Deployment pattern:** One **main weather mast** near operations center; multiple **field nodes** distributed across a large property (intentionally vague for public repo).
- **Aesthetics & safety:** Not unsightly; not a hazard to livestock/equipment; no long cable runs across property.

---

## 3) High‑Level System Overview
- **Core hub:** Raspberry Pi (RPi 4/5) hosting data ingestion, local database, APIs, and dashboard (LAN‑only by default; offline‑first).
- **Remote nodes:** Low‑power microcontroller nodes (ESP32/Arduino class) sending sensor data to hub over **LoRa (915 MHz)**; near‑hub nodes may use Wi‑Fi. No property‑spanning wired runs.
- **Data model:** Raw samples → minute aggregates → hourly/daily rollups, with metadata for calibration and maintenance.
- **Optional cloud export:** One‑way periodic sync to a private cloud dashboard when internet is available; local system remains fully functional without internet.

**Architecture sketch (textual):**  
Sensors → Node MCU (ESP32) → LoRa uplink → LoRa gateway @ RPi hub → Ingestion service → Time‑series DB → API → Local Dashboard

---

## 4) Power Strategy
- **Primary:** Solar + LiFePO₄ battery on mast/field nodes; sized for ≥3 days autonomy and peak summer load.
- **Hub power:** Mains with small UPS (graceful shutdown + brief outages).
- **Fallbacks:** Small nodes may use alternative local power sources where solar is impractical; all nodes include battery backup.

---

## 5) Sensor Set & MVP
### 5.1 Main Weather Mast (MVP)
- **Wind:** speed & direction
- **Rain:** tipping bucket
- **Air:** temperature, relative humidity (RH), barometric pressure
- **Radiation:** solar irradiance + UV index
- **Lightning:** strike count/approx. distance (event‑based)
- **(Optional on mast):** Soil surface temperature & moisture

### 5.2 Field/Ecosystem Nodes (post‑MVP)
- **Water trough monitor:** temperature, level, freeze/dry/overflow detection
- **Other environmental nodes** as needed (LoRa preferred; Wi‑Fi near hub)

---

## 6) Engineering Targets (Initial)
(*To be validated during sensor selection & bench tests*)
- **Air temperature:** ±0.3–0.5 °C (−30 … +55 °C operating)
- **Relative humidity:** ±2 % RH (10–90 %), ±3 % full range
- **Pressure:** resolution ≤0.1 hPa, absolute accuracy ±0.3–0.5 hPa, repeatability ≤±0.1–0.2 hPa, drift <±1 hPa/year
- **Wind speed:** ±(0.3 m/s or 3 %), start threshold <0.5–0.8 m/s
- **Wind direction:** ±5° (clean siting), ±10° (typical ranch siting)
- **Rain gauge:** resolution 0.2 mm per tip; cumulative accuracy ±4–5 %
- **Solar irradiance:** ±5–10 % of reading
- **UV index:** ±0.5 UVI (typical sensor limits)
- **Soil moisture (VWC):** resolution ≤2 % with temperature compensation
- **Lightning:** event detection with approximate distance (qualitative)

---

## 7) Data Cadence, Retention & Aggregation (Proposed)
### 7.1 Sampling Cadence
- **Wind:** sample at 1 s; compute 1‑min average, gust (max 3‑s), and direction distribution
- **Rain:** event‑driven (on tip); also report 1‑min, 1‑h totals
- **Air T/RH/Pressure:** every 30 s
- **Solar/UV:** every 30 s
- **Lightning:** event‑driven with per‑minute count & nearest‑strike estimate
- **Soil (temp/moisture):** every 60 s (field nodes may be slower to save power)

### 7.2 On‑Device Retention
- **Node (MCU):** ring buffer for ≥7 days of 1‑min aggregates; opportunistic backfill if gateway link is down
- **Hub (RPi):**
  - Raw (≤30 s data): 7 days
  - 1‑min aggregates: 90 days
  - 15‑min aggregates: 1 year
  - 1‑h aggregates: 2+ years

*(Durations configurable via environment settings.)*

---

## 8) Networking & Protocols
- **Primary link:** LoRa (915 MHz, US) from nodes to hub gateway
- **Near‑hub link:** Wi‑Fi for bench/adjacent nodes
- **Protocol:** lightweight binary or CBOR over LoRa; MQTT internally on hub (optional); HTTPS/REST or WebSocket for dashboard/API
- **Time sync:** hub maintains time; nodes sync periodically via gateway; tolerant of long offline periods
- **Security:** LAN‑only by default; API keys for local services; no secrets in repo

---

## 9) Physical Design & Reliability
- **Mast:** 10–12 ft grounded pole; guy‑wires if needed; compliant lightning protection
- **Enclosures:** UV‑stable IP65+ polycarbonate, breathable membrane for pressure/condensation, insect mesh
- **Connectors:** weather‑rated (IP67) with drip loops; strain relief; service loops
- **PCB/assembly:** conformal coating where appropriate; corrosion‑resistant fasteners
- **Maintenance:** tool‑accessible mounting, swap‑friendly sensor modules, clear labeling & docs

---

## 10) Software/Repo Boundaries
- **Local‑only by default** (core requirement). Optional module may handle cloud export.
- **Repo structure:**
  - `/docs/` — planning, diagrams, test plans, calibration
  - `/firmware/` — MCU code (ESP32/Arduino)
  - `/hub/` — Raspberry Pi services (ingest, DB, API, dashboard)
  - `/hardware/` — BOM, schematics, wiring, enclosure & mast notes
  - `/tests/` — validation scripts and procedures

---

## 11) Deliverables
- **D1:** Scope document and architecture diagram
- **D2:** Hardware inventory & BOM; sensor evaluation matrix
- **D3:** Bench prototype with ingestion → dashboard path
- **D4:** Outdoor mast prototype with solar/battery power
- **D5:** Firmware alpha
- **D6:** Dashboard MVP
- **D7:** Field node (e.g., water trough) v1 + alerting
- **D8:** v1.0 release with install guide, calibration notes, maintenance SOP, safety

---

## 12) Milestones
- **v0.1 – Scope & Repo**
- **v0.2 – Bench Rig**
- **v0.3 – Power & Enclosure**
- **v0.4 – Mast Deploy**
- **v0.5 – LoRa Field Node**
- **v0.6 – Data Quality**
- **v0.7 – Dashboard MVP**
- **v1.0 – Field‑tested**

---

## 13) Validation & Calibration Plan
- Side‑by‑side test of T/RH/pressure with a reference sensor
- Anemometer spin test; direction calibration via known headings
- Rain gauge calibration with measured pour volume
- Solar/UV check vs clear‑day curves; shading analysis
- Soil probe calibration with known moisture media
- Lightning sensor lab sanity, field observation during storms

---

## 14) Risks & Mitigations
- **Lightning/Surge:** grounding, surge arrestors, isolated power, replaceable modules
- **Humidity ingress:** IP65+ enclosures, vents, desiccant packs, conformal coat
- **Wildlife interference:** screened vents, cable armor where exposed
- **Power budget:** duty‑cycling, low‑power modes, right‑size panels/batteries
- **Radio range:** antenna placement, gateway height, LoRa spreading factor tuning

---

## 15) Open Decisions / TBD
- Sensor models and vendors
- Final sampling/aggregation per sensor after bench data
- Hub DB choice (SQLite/Timescale/Influx)
- Cloud export mechanism
- Mast location/height after site survey

---

## 16) Non‑Goals (v1)
- Public internet exposure of the hub
- Dependence on cloud for core operations
- Long property‑spanning wired runs

---

### Document Control
- **Filename:** `docs/01_scope.md`  
- **Future versions:** For major changes, create `docs/02_scope.md` (and so on) to preserve history. Latest version should be linked from `README.md`.