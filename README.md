
# 🔥 3-Phase Heater Controller

## 📘 Overview
This Node-RED flow controls **three heating relays (L1, L2, L3)** connected to Victron VE.Bus based systems.
It intelligently uses **excess PV energy** to heat water while protecting the inverter, battery, and boiler.

### ⚡ Key Features
- 3-phase relay control (L1–L3)
- Uses Victron **battery power, SOC, AC loads** for logic
- **Progressive ON/OFF** switching — one relay per step
- **Flat hysteresis** ±1000 W (no chatter)
- Heating only when:
  - Battery charging
  - SOC ≥ min SOC
  - Within sunrise–sunset window
  - Boiler < 40 °C
  - Inverter load ≤ 5000 W
- **Manual Override** switch → force all relays ON (emergency heating)
- **Failsafe timers** (min ON/OFF durations)
- **Throttle + change detection** for relay updates and status
- Fully configurable from **Node-RED Dashboard**

---

## 🧩 System Architecture
```
[Victron VE.Bus Inputs] ──► [Heater Controller Function Node]
│
├─ Battery Power
├─ SOC
├─ AC Load (L1–L3)
├─ Boiler Temperature
├─ Inverter Output Load
│
▼
[Dashboard UI]
├─ Config Form (parameters)
├─ Manual Override Switch
└─ Status Panel (SOC, Temp, Relays, Load)

Outputs:
/Relay/0/State  → L1
/Relay/1/State  → L2
/Relay/2/State  → L3
/Heater/Status  → status object
```

---

## 🧱 Installation

### 1. Requirements
- Node-RED ≥ 3.x
- `node-red-dashboard`
- `node-red-contrib-victron`

### 2. Import Flow
1. Copy the provided `.json` flow file.
2. In Node-RED → **Menu → Import → Clipboard → Paste JSON** → Import.
3. Deploy the flow.

### 3. Connect Victron Nodes
Wire:
- `VE.Bus Battery Power`
- `VE.Bus SOC`
- `VE.Bus AC Loads (L1–L3)`
- `VE.Bus Boiler Temp Sensor`
- `VE.Bus AC Output Load`
to the **Heater Controller** function node.

### 4. Dashboard Tabs
- **Heater Control:** configuration form
- **Heater Monitor:** status panel + manual override switch

---

## ⚙️ Configurable Parameters

| Parameter | Default | Description |
|------------|----------|-------------|
| `onThreshold` | +1000 W | Start heating when battery charging above this |
| `offThreshold` | −1000 W | Stop heating when discharging below this |
| `minSoc` | 70 % | SOC threshold to allow heating |
| `boilerTempMax` | 40 °C | Stop if boiler too hot |
| `inverterLoadMax` | 5000 W | Block heating if inverter overloaded |
| `minOnSec` | 60 | Minimum relay ON time |
| `minOffSec` | 60 | Minimum relay OFF time |
| `minAnyChangeSec` | 10 | Global cooldown between relay changes |
| `statusMinIntervalSec` | 5 | Throttle status messages |
| `manualTimeoutSec` | 600 | Auto-release manual override (seconds) |

Values adjustable at runtime via Dashboard → Heater Control tab.

---

## 🧪 Operation Logic

1. **Collect Victron data** (Battery, SOC, Loads, Temp, etc.)
2. Evaluate conditions:
   - If PV surplus → progressively turn relays ON (lowest load first)
   - If deficit → progressively turn relays OFF (highest load first)
   - Skip ON sequence if inverter load > limit or SOC < limit
3. Enforce hysteresis ±1000 W
4. Apply relay min-ON/OFF times and global cooldown
5. Output only changed states
6. Send throttled `/Heater/Status` message for dashboard and debug
7. If **manual override ON** → all relays forced ON, logic paused

---

## 🔍 Debug & Monitoring
- **Debug Node** (connected to status output): shows object with live state.
- **Dashboard → Heater Monitor:** displays
```
  SOC: 74 %
  Boiler Temp: 38 °C
  Inverter Load: 3400 W
  Relays: [1, 1, 0]
  Mode: AUTO
```
- Manual override turns mode to `MANUAL: FORCE ON`.

---

## 🧰 Maintenance Notes
- For mechanical relays, use ≥ 60 s ON/OFF times to avoid wear.
- For SSRs, 10–15 s progression cooldown is recommended.
- Test thoroughly with relays disconnected before live deployment.
- Verify phase load balancing (L1–L3) matches Victron readings.
- The flow retains its internal state after redeploy (uses `context`).

---

## 🧩 Topics Summary

| Input | Purpose |
|-------|----------|
| `/Dc/Battery/Power` | Battery power (W) |
| `/Dc/Battery/Soc` | Battery SOC (%) |
| `/Ac/L1/Power` | Line 1 load (W) |
| `/Ac/L2/Power` | Line 2 load (W) |
| `/Ac/L3/Power` | Line 3 load (W) |
| `/Boiler/Temp` | Boiler temperature (°C) |
| `/Inverter/Load` | Inverter AC output load (W) |
| `/Heater/Config` | Runtime configuration update |
| `/Heater/Manual` | Manual override toggle (1 = ON, 0 = OFF) |

| Output | Description |
|--------|--------------|
| `/Relay/0/State` | Relay L1 (0/1) |
| `/Relay/1/State` | Relay L2 (0/1) |
| `/Relay/2/State` | Relay L3 (0/1) |
| `/Heater/Status` | JSON object with full controller status |

---

## 📎 Example Status Object
```json
{
  "relays": [1, 1, 0],
  "soc": 74,
  "batteryPower": 2200,
  "boilerTemp": 37,
  "inverterLoad": 3400,
  "manualOverride": false,
  "mode": "AUTO",
  "lastChange": "2025-10-06T13:42:10Z"
}
```

⸻

🧩 Known Good Defaults

```json
{
  "onThreshold": 1000,
  "offThreshold": -1000,
  "minSoc": 70,
  "boilerTempMax": 40,
  "inverterLoadMax": 5000,
  "minOnSec": 60,
  "minOffSec": 60,
  "minAnyChangeSec": 10,
  "manualTimeoutSec": 600,
  "statusMinIntervalSec": 5
}
```

⸻

### 🏁 Credits
Developed collaboratively in 2025-10 by Code Copilot & Larry0t (Victron & Node-RED enthusiast).  
Optimized for VE.Bus relay control, safe PV surplus management, and extendable logic (multiple heaters, EV chargers, etc.).

⸻
