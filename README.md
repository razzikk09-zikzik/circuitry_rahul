# Circuitry

**Circuitry** is a full-stack, rule-based circuit design tool for electronics and embedded systems. Pick a domain, configure your design, and generate wiring diagrams, netlists, Arduino code, bills of materials, and build instructions — all deterministically, with no external AI API required.

---

## Features

- **12 design domains** — microcontroller systems, logic gates, analog/digital filters, passive networks, power electronics, protection, communication interfaces, timing circuits, and natural-language synthesis (NLM)
- **Interactive schematic diagrams** — SVG-based wiring views with protocol-aware wire colors
- **Auto pin allocation** — I2C/SPI/UART bus sharing, PWM/analog assignment, ESP32 ADC1 vs ADC2 handling
- **Support component injection** — auto-adds resistors, transistors, flyback diodes, motor drivers per rules
- **Arduino/ESP code generation** — per-component `.ino` templates with pin definitions
- **Validation engine** — voltage compatibility, pin conflicts, current draw, I2C address clashes
- **Multi-tab output** — Info, Diagram, Parts, Code, Instructions, Netlist, Power, PCB, BOM
- **Project save/load** — persist configs and results to local JSON files
- **Deterministic & offline-capable** — all logic runs locally; no cloud LLM dependency

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React 18, Vite 5, Tailwind CSS, Framer Motion |
| Backend | Node.js, Express 4 |
| Data | JSON files (components, platforms, rules, trained circuits) |
| Storage | File-based project persistence (`projects/`) |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  React Client (port 5173)                                   │
│  InputForm → useCircuitGenerator → api.js                   │
│  App → CircuitDiagram | CodeViewer | NetlistViewer | BOM…   │
└──────────────────────────┬──────────────────────────────────┘
                           │ POST /api/generate
┌──────────────────────────▼──────────────────────────────────┐
│  Express API (port 3001/3002)                               │
│  routes/circuit.js → domain-specific engine                 │
└──────────────────────────┬──────────────────────────────────┘
                           │
     ┌─────────────────────┼─────────────────────┐
     │                     │                     │
 ruleEngine          booleanBuilder         nlmEngine
 (microcontroller)   analogFilterBuilder    powerBuilder
     │                electricBuilder        …etc
     ▼
 pinAllocator → circuitBuilder → codeGenerator → validator
```

### Microcontroller pipeline

The primary (most complete) generation path:

1. **Resolve platform** — load pin map from `server/data/platforms.json`
2. **Resolve components** — load selected parts from `server/data/components.json`
3. **Feature rules** — e.g. IoT feature suggests WiFi-capable modules
4. **Support rules** — e.g. relay → transistor + flyback diode
5. **Duplicate handling** — suffix instance IDs (`dht11_2`, `dht11_3`)
6. **Pin allocation** — assign MCU pins by protocol (`pinAllocator.js`)
7. **Circuit build** — connection graph + JSON netlist (`circuitBuilder.js`)
8. **Code generation** — Arduino/ESP `.ino` output (`codeGenerator.js`)
9. **Validation** — errors, warnings, info (`validator.js`)
10. **Explanation** — markdown overview of the design

---

## Supported Domains

| Domain ID | Engine | Description |
|-----------|--------|-------------|
| `microcontroller` | `ruleEngine.js` | Arduino, ESP32, Pico — sensors, actuators, displays, comm modules |
| `boolean_logic` | `booleanBuilder.js` | Boolean expressions → gate diagrams and truth tables |
| `analog_filter` | `analogFilterBuilder.js` | RC/RLC analog filter design |
| `digital_filter` | `digitalFilterBuilder.js` | DSP filters, Z-transform analysis |
| `electric` | `electricBuilder.js` | Passive network topologies (voltage dividers, etc.) |
| `passive` | `passiveBuilder.js` | R/L/C networks, dividers, impedance matching |
| `analog` | `analogBuilder.js` | Op-amps, BJT/MOSFET, comparators |
| `power` | `powerBuilder.js` | LDOs, buck/boost converters, battery chargers |
| `protection` | `protectionBuilder.js` | ESD, overcurrent, overvoltage protection |
| `communication` | `commBuilder.js` | RS232, I2C, SPI, CAN, level shifters |
| `timing` | `timingBuilder.js` | 555 timers, oscillators, clock circuits |
| `nlm` | `nlmEngine.js` | Natural-language query → architecture from trained DB |

---

## Project Structure

```
circuitry/
├── client/                    # React frontend
│   ├── src/
│   │   ├── App.jsx            # Main layout + tab routing
│   │   ├── components/
│   │   │   ├── InputForm.jsx      # Multi-step config wizard
│   │   │   ├── CircuitDiagram.jsx # SVG schematic renderer
│   │   │   ├── CodeViewer.jsx     # Syntax-highlighted code
│   │   │   ├── ComponentList.jsx  # Parts list with pricing
│   │   │   ├── NetlistViewer.jsx  # JSON netlist viewer
│   │   │   ├── ExplanationPanel.jsx
│   │   │   ├── Instructions.jsx   # Build steps
│   │   │   ├── PowerAnalysis.jsx
│   │   │   ├── PCBGuide.jsx
│   │   │   ├── BOMExport.jsx
│   │   │   ├── ProjectManager.jsx
│   │   │   ├── LandingPage.jsx
│   │   │   └── Header.jsx
│   │   ├── hooks/
│   │   │   └── useCircuitGenerator.js  # Config state + API calls
│   │   └── utils/
│   │       ├── api.js             # Fetch wrappers
│   │       └── circuitRenderer.js # Diagram layout + symbols
│   └── vite.config.js         # Dev server + API proxy
│
├── server/                    # Express backend
│   ├── index.js               # Server entry point
│   ├── routes/
│   │   ├── circuit.js         # Generate, validate, components, platforms
│   │   └── projects.js        # CRUD for saved projects
│   ├── engine/
│   │   ├── ruleEngine.js      # Microcontroller orchestrator
│   │   ├── pinAllocator.js    # MCU pin assignment
│   │   ├── circuitBuilder.js  # Connection graph + netlist
│   │   ├── codeGenerator.js   # Arduino/ESP code templates
│   │   ├── validator.js       # Circuit validation
│   │   ├── booleanBuilder.js
│   │   ├── analogFilterBuilder.js
│   │   ├── digitalFilterBuilder.js
│   │   ├── electricBuilder.js
│   │   ├── passiveBuilder.js
│   │   ├── analogBuilder.js
│   │   ├── powerBuilder.js
│   │   ├── protectionBuilder.js
│   │   ├── commBuilder.js
│   │   ├── timingBuilder.js
│   │   └── nlmEngine.js       # Natural language matcher
│   ├── data/
│   │   ├── components.json    # ~89 component definitions
│   │   ├── platforms.json     # MCU platform pin maps
│   │   ├── rules.json         # Feature + support component rules
│   │   └── trained_circuits.json  # NLM architecture database
│   └── scripts/
│       ├── validate_data.js
│       └── test_engine_suites.js
│
├── projects/                  # Saved user projects (JSON, gitignored if local)
├── package.json               # Root scripts (dev, install:all)
├── .env.example               # Environment template
└── README.md
```

---

## Getting Started

### Prerequisites

- Node.js 18+
- npm

### Installation

```bash
git clone https://github.com/rahulnerella86/circuitry.git
cd circuitry
npm run install:all
```

### Configuration

Copy the environment template and adjust if needed:

```bash
cp .env.example .env
```

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3001` | Express API server port |

The Vite dev server proxies `/api` requests to the backend (configured in `client/vite.config.js`).

### Run (development)

```bash
npm run dev
```

| Service | URL |
|---------|-----|
| Frontend | http://localhost:5173 |
| API | http://localhost:3001 (or your `PORT` in `.env`) |

### Run (production)

```bash
# Build frontend
cd client && npm run build

# Start API server
cd ../server && npm start
```

Serve `client/dist/` with any static file server, pointing API calls to the Express backend.

---

## API Reference

### Circuit generation

```
POST /api/generate
Content-Type: application/json

{
  "domain": "microcontroller",
  "platform": "esp32",
  "sensors": ["dht11"],
  "actuators": ["relay"],
  "displays": [],
  "communication": [],
  "discretes": [],
  "power": [],
  "features": ["iot"]
}
```

### Other endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/validate` | Validate config without full generation |
| `GET` | `/api/components` | List all components (`?category=sensor`) |
| `GET` | `/api/platforms` | List supported MCU platforms |
| `GET` | `/api/health` | Health check |
| `GET` | `/api/projects` | List saved projects |
| `POST` | `/api/projects` | Save a project |
| `GET` | `/api/projects/:id` | Load a project |
| `PUT` | `/api/projects/:id` | Update a project |
| `DELETE` | `/api/projects/:id` | Delete a project |

---

## Supported Platforms

| Platform ID | Board |
|-------------|-------|
| `arduino_uno` | Arduino Uno (ATmega328P) |
| `arduino_nano` | Arduino Nano |
| `arduino_mega` | Arduino Mega 2560 |
| `esp32` | ESP32 DevKit |
| `esp8266` | ESP8266 (NodeMCU/Wemos) |
| `rpi_pico` | Raspberry Pi Pico |

Each platform defines pin capabilities (digital, analog, PWM, I2C, SPI, UART), reserved pins, bus mappings, voltage, and current limits.

---

## Component Library

Components in `server/data/components.json` are organized by category:

- **sensor** — DHT11/22, LDR, MQ-2, HC-SR04, MPU6050, BMP280, PIR, etc.
- **actuator** — LED, relay, DC motor, servo, buzzer, stepper
- **display** — LCD I2C, OLED SSD1306, 7-segment
- **communication** — HC-05 Bluetooth, NRF24L01, SD card
- **discrete** — buttons, switches
- **power** — external power modules

Each component includes pins, protocol (`i2c`, `spi`, `uart`, `analog`, `pwm`, `digital`, `oneWire`), voltage range, I2C address, and optional supporting components.

---

## Rules Engine

`server/data/rules.json` defines three rule types:

### Feature rules
Add suggested components when a feature is enabled (e.g. IoT → WiFi module if platform supports it).

### Support component rules
Auto-add required passives and drivers:
- Relay → 2N2222 transistor, 1kΩ base resistor, 1N4007 flyback diode
- DC motor → L293D H-bridge, decoupling capacitor
- And more per component type

### Pin assignment rules
Document pin allocation requirements (analog → ADC, I2C → shared bus, SPI → unique CS). The actual allocation logic lives in `pinAllocator.js`.

---

## Result Schema

All domain engines return a unified object consumed by the UI:

```json
{
  "platform": { "id": "esp32", "name": "ESP32 DevKit", "operatingVoltage": 3.3 },
  "components": [
    { "id": "dht11", "name": "DHT11", "category": "sensor", "voltage": { "min": 3, "max": 5.5, "typical": 3.3 } }
  ],
  "pinAssignments": [
    { "componentId": "dht11", "componentName": "DHT11", "protocol": "digital", "pins": { "DATA": { "pinId": "GPIO4", "pinName": "D4" } } }
  ],
  "connections": [
    { "id": "conn_1", "from": { "node": "node_dht11", "pin": "DATA" }, "to": { "node": "node_mcu", "pin": "GPIO4" }, "type": "signal", "color": "#6366f1" }
  ],
  "netlist": {
    "version": "1.0",
    "platform": "esp32",
    "nets": { "VCC": { "name": "VCC", "type": "power", "connections": [] } },
    "statistics": { "totalNets": 5, "totalConnections": 8, "totalNodes": 4 }
  },
  "code": "// Arduino .ino file contents…",
  "explanation": "## Circuit Overview\n\n…",
  "validation": {
    "valid": true,
    "errors": [],
    "warnings": ["Estimated current draw is 70% of platform capacity."],
    "info": [],
    "summary": "⚠️ Circuit is valid with 1 warning(s)."
  }
}
```

---

## Client UI

The app uses a split-panel layout:

- **Left panel** — `InputForm` multi-step wizard (domain → platform → components → features)
- **Right panel** — tabbed output once a circuit is generated

| Tab | Component | Content |
|-----|-----------|---------|
| Info | `ExplanationPanel` | Markdown explanation + validation results |
| Diagram | `CircuitDiagram` | SVG schematic with legend |
| Parts | `ComponentList` | Component list with editable prices |
| Code | `CodeViewer` | Syntax-highlighted `.ino` or truth table |
| Instructions | `Instructions` | Step-by-step build checklist |
| Netlist | `NetlistViewer` | JSON netlist with statistics |
| Power | `PowerAnalysis` | Current draw and thermal notes |
| PCB | `PCBGuide` | Layout recommendations |
| BOM | `BOMExport` | Bill of materials export |

State is managed by the `useCircuitGenerator` hook, which handles config, validation, API calls, and tab state.

---

## NLM (Natural Language Mode)

The `nlm` domain uses `nlmEngine.js` to match user queries against pre-built architectures in `trained_circuits.json` via keyword scoring (categories: power, fpga, analog, mixed-signal). It returns architecture diagrams, HDL snippets where applicable, and an AI-style explanation. This is a local, deterministic matcher — not a cloud LLM.

---

## Development

### Scripts

```bash
npm run dev          # Start client + server concurrently
npm run dev:client   # Frontend only
npm run dev:server   # Backend only
npm run install:all  # Install root, server, and client dependencies
```

### Data validation

```bash
node server/scripts/validate_data.js
node server/scripts/test_engine_suites.js
```

### Adding a new component

1. Add an entry to `server/data/components.json` with pins, protocol, and voltage
2. Add support rules to `server/data/rules.json` if needed
3. Add a code template in `server/engine/codeGenerator.js` (`generateComponentCode` switch)
4. Run validation scripts

### Adding a new platform

1. Add a full pin map to `server/data/platforms.json`
2. Include I2C/SPI/UART bus definitions, reserved pins, and current limits
3. Test pin allocation with multiple component combinations

---

## Design Philosophy

Circuitry is intentionally **deterministic and rule-based**:

- Every output is reproducible from the same input
- No external API keys or cloud services required
- Component selection, pin assignment, and code generation follow explicit, inspectable rules
- The JSON data files are the source of truth and can be extended without changing core engine logic

---

## License

Private project. All rights reserved.

---

## Author

[Rahul Nerella](https://github.com/rahulnerella86)
