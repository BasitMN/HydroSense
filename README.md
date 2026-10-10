# Hydrogreen
## AI-powered plant threshold logic for MicroHydros
## IoT- och embeddedprototyp

MicroHydros is an IoT prototype that monitors a small growing environment (e.g. hydroponics). An ESP32-S3 sensor node reads internal air temperature/humidity (SHT31) and water and external air temperature (two DS18B20 probes), and publishes one raw MQTT telemetry message every 5 seconds. A Docker-based backend (Mosquitto, Node-RED, InfluxDB, Grafana) validates, stores and displays the measurements. See [`docs/architecture.md`](docs/architecture.md) for the full system design and [`docs/data-contract.md`](docs/data-contract.md) for the exact MQTT/JSON contract.

## About this copy

This is a personal copy of a team project built for the "IoT- och embeddedutvecklare" program at Jensen YH (project period 31 Aug – 25 Sep 2026), kept here for my own portfolio. The active, canonical repository — with the full team, ongoing issues and pull requests — is **[Samuel2Nugor/Hydrogreen](https://github.com/Samuel2Nugor/Hydrogreen)**. Every commit here still carries its original author, so this history shows the whole team's work, not only mine; see "My contribution" below for what I specifically did.

## My contribution (Basit MN)

The team was five people. By commit count I was the most active contributor (24 of 69 commits on `main`, roughly 1,950 lines added and 620 removed), close to [Samuel2Nugor](https://github.com/Samuel2Nugor) at 23. My work concentrated on three areas:

* **ESP32-S3 firmware** ([`firmware/esp32s3/`](firmware/esp32s3/)) — the ESP-IDF application: Wi-Fi, MQTT client and the SHT31/DS18B20 sensor drivers, including ROM-address pinning so the two identical DS18B20 probes can share one 1-Wire bus without their readings silently swapping. Later split from a single `main.c` into separate Wi-Fi/MQTT/sensor modules in [PR #18](https://github.com/Samuel2Nugor/Hydrogreen/pull/18), in response to code-review feedback on [PR #16](https://github.com/Samuel2Nugor/Hydrogreen/pull/16).
* **Backend deployment** — brought the whole Docker stack (Mosquitto, Node-RED, InfluxDB, Grafana) up on a shared Hetzner server for the team demo, including a port remap to coexist with another service on the same host, Node-RED authentication and automatic Grafana provisioning ([PR #16](https://github.com/Samuel2Nugor/Hydrogreen/pull/16)).
* **Documentation and testing** — architecture, sensor/hardware justification, hardware and firmware READMEs, and the executed test results in [`docs/test-plan.md`](docs/test-plan.md).

One concrete example: [issue #14](https://github.com/Samuel2Nugor/Hydrogreen/issues/14) — after the external sensor was renamed from `external_sht31` to `external_ds18b20` elsewhere in the codebase, the Node-RED validation flow still referenced the old name and silently rejected every outdoor-temperature reading. I found the root cause, wrote it up with a proposed fix and a verification step, and closed it with the corresponding commit.

## Repository structure

| Path | Purpose |
|------|---------|
| `docs/` | Project planning and technical documentation |
| `firmware/` | ESP32-S3 firmware (ESP-IDF, C) |
| `hardware/` | Bill of materials, pinout and wiring documentation |
| `docker/` | Backend services (Mosquitto, Node-RED, InfluxDB, Grafana) via Docker Compose |
| `src/` | Source code for the prototype (e.g. the telemetry simulator) |
| `tests/` | Tests and supporting test code — see [`docs/test-plan.md`](docs/test-plan.md) |
| `data/` | Small sample measurements and example data formats |
| `CONTRIBUTING.md` | Rules for branches, commits, reviews and pull requests |

The structure may change as the system architecture becomes clearer.

## Dependencies

* Docker Engine + Docker Compose — runs the backend (see [`docker/README.md`](docker/README.md))
* [ESP-IDF v5.1+](https://docs.espressif.com/projects/esp-idf/en/v5.1.2/esp32s3/get-started/index.html) — builds and flashes the firmware (see [`firmware/esp32s3/README.md`](firmware/esp32s3/README.md))
* Python 3 — for the telemetry simulator (see [`src/simulator/README.md`](src/simulator/README.md)) and tooling bundled with ESP-IDF

## Quick start

1. **Start the backend.** `cd docker && docker compose up -d` — see [`docker/README.md`](docker/README.md) for `.env` setup and health checks.
2. **Get telemetry flowing**, either:
   * Run the simulator (no hardware needed): see [`src/simulator/README.md`](src/simulator/README.md), or
   * Flash the real ESP32-S3: see [`hardware/esp32s3/README.md`](hardware/esp32s3/README.md) for wiring and [`firmware/esp32s3/README.md`](firmware/esp32s3/README.md) for build/flash/menuconfig.
3. **Import the Node-RED flows** (`docker/node-red/flows/`) if not already provisioned — see [`docker/README.md`](docker/README.md).
4. **Check it's working:**
   * Node-RED editor: `http://localhost:1880` (or the deployment's mapped port)
   * Grafana dashboard: `http://localhost:3000` — shows current and historical measurements once valid data has been written to InfluxDB
   * Raw MQTT: subscribe to `microhydros/v1/devices/+/telemetry/raw` to see incoming device messages; `.../telemetry/validated/+` for accepted measurements and `.../telemetry/rejected` for anything Node-RED rejected (see [`docs/architecture.md`](docs/architecture.md) § MQTT topics)

For the team's demonstration deployment (Hetzner, not a laptop), see [`docs/architecture.md`](docs/architecture.md) § Demonstration environment for the actual addresses and ports.
