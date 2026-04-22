# EOS Swarm GCS

A browser-based Ground Control Station for managing a swarm of drones using the MAVLink v2 protocol. Built for real field operations — dark theme, instant status-at-a-glance, safety confirmations on all destructive commands, and keyboard shortcuts for rapid response.

---

## Architecture

```
[Drone 1] ──UDP:14550──┐
[Drone 2] ──UDP:14551──┤──► [FastAPI + pymavlink Bridge] ──WS──► [React App]
[Drone N] ──UDP:14552──┘         (one thread per drone)           (Vite + Zustand)
```

The Python backend listens on one UDP port per drone, parses MAVLink v2 messages, and relays them as JSON over WebSocket. A watchdog thread broadcasts a `REGISTRY_SNAPSHOT` every second so the frontend always reflects the latest drone status — including signal loss.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 + Vite |
| State | Zustand |
| Map | Leaflet + react-leaflet |
| Styling | Tailwind CSS |
| Charts | recharts |
| Backend | Python FastAPI + uvicorn |
| MAVLink | pymavlink |
| Icons | lucide-react |

---

## UI Layout

```
┌─────────────────────────────────────────────────────────────────┐
│ TOP BAR: [● WS Connected]  Swarm: 3/3 OK  [⚠ Drone 2 Low Batt] │
├────────────┬────────────────────────────────┬────────────────────┤
│            │                                │  [Telemetry]       │
│  FLEET     │                                │  [Commands]        │
│  SIDEBAR   │        MAP (Leaflet)           │  [Mission]         │
│            │                                │  [Log]             │
│  Drone 1 ● │   ✈ ✈ ✈  (blips + arrows)     │                    │
│  Drone 2 ⚠ │                                │  ← context panel   │
│  Drone 3 ● │   click drone → select         │    switches on     │
│            │                                │    drone select    │
│  [ARM ALL] │                                │                    │
│  [LAND ALL]│                                │                    │
│  [RTL ALL] │                                │                    │
│  [E-STOP]  │                                │                    │
└────────────┴────────────────────────────────┴────────────────────┘
```

**Design principles:**
- Map is always center — never hidden behind tabs
- Alert banner appears only when there is an active warning
- E-STOP is always visible, one click + confirmation modal away
- Right panel auto-switches to Telemetry when a drone is selected
- Dark theme throughout for outdoor readability

---

## Features

### Fleet Sidebar
- Live drone list with status badges: `ARMED` · `IDLE` · `RTL` · `LOST`
- Battery bar and signal indicator per drone
- Swarm broadcast buttons: ARM ALL, LAND ALL, RTL ALL, E-STOP
- Click any drone to select and focus the right panel

### Map View
- Dark tile layer (CartoDB Dark Matter)
- SVG drone markers with heading arrows, color-coded by status
- Pulse animation on WARNING / LOST drones
- Mission waypoint polyline overlay
- Geofence polygon with transparent fill

### Telemetry Panel
- LAT / LON / ALT · Speed · Heading · Battery % · Flight mode · Armed state
- Artificial horizon (SVG attitude indicator from ATTITUDE roll/pitch)
- 60-second altitude sparkline

### Command Panel
- Per-drone: ARM · DISARM · TAKEOFF (altitude input) · LAND · RTL · HOLD · RESUME
- Swarm: ARM ALL · LAND ALL · RTL ALL · EMERGENCY STOP
- Confirmation modal required for ARM, DISARM, and all swarm commands

### Mission Planning
- Click map to place waypoints
- Drag to reorder in Mission panel
- Upload to drone via MAVLink `MISSION_ITEM_INT` stream
- Formation presets: Grid, Line

### Safety System
| Condition | Action |
|---|---|
| No HEARTBEAT for 3 s | Status → LOST, alert banner |
| Battery < 20% | Warning alert, badge flashes |
| Battery < 10% | Auto-RTL (configurable) |
| `STATUSTEXT` severity ≤ 3 | Critical alert banner |
| Geofence breach | Warning alert |
| E-STOP | Disarm all — requires confirmation |

### Keyboard Shortcuts
| Key | Action |
|---|---|
| `Esc` | Emergency stop (opens confirm) |
| `Ctrl+L` | Land all |
| `Ctrl+R` | RTL all |
| `1` – `9` | Select drone by index |
| `Tab` | Cycle right panel tabs |

### Video Feed (RTSP)

Auterion OS drones expose camera video over RTSP. The GCS bridges each stream to the browser using FFmpeg (RTSP → HLS) and plays it with `hls.js`.

```
Drone RTSP :8554/live  →  FFmpeg (backend)  →  HLS  →  hls.js (browser)
```

**Selecting a stream:**
1. Open the **Video** tab in the right panel
2. Choose a drone from the dropdown — if the drone broadcast `VIDEO_STREAM_INFORMATION`, the RTSP URL is pre-filled automatically
3. Edit the URL if needed (default pattern: `rtsp://<drone-ip>:8554/live`)
4. Click **Connect** — the backend starts an FFmpeg process and the player begins buffering
5. Click **Disconnect** to stop the stream and free the FFmpeg process

Each drone gets its own independent stream process. Switching drones disconnects the current stream automatically.

**Requires FFmpeg installed on the backend host:**
```bash
# Ubuntu/Debian
sudo apt install ffmpeg
# macOS
brew install ffmpeg
```

---

## MAVLink Messages Handled

| Message | Fields |
|---|---|
| `HEARTBEAT` | `custom_mode`, `system_status`, `base_mode` |
| `GLOBAL_POSITION_INT` | `lat`, `lon`, `alt`, `relative_alt`, `hdg` |
| `SYS_STATUS` | `battery_remaining`, `voltage_battery` |
| `ATTITUDE` | `roll`, `pitch`, `yaw` |
| `VFR_HUD` | `airspeed`, `groundspeed`, `alt`, `climb` |
| `GPS_RAW_INT` | `fix_type`, `satellites_visible` |
| `MISSION_CURRENT` | `seq` |
| `STATUSTEXT` | `severity`, `text` |
| `VIDEO_STREAM_INFORMATION` | `uri` (RTSP URL auto-populated) |

---

## Project Structure

```
GGS/
├── backend/
│   ├── main.py               # FastAPI app, WebSocket endpoint
│   ├── mavlink_bridge.py     # Per-drone UDP listeners + broadcast
│   ├── drone_registry.py     # Telemetry state + heartbeat watchdog
│   ├── command_handler.py    # MAVLink command dispatcher
│   └── requirements.txt
└── frontend/
    ├── src/
    │   ├── App.jsx
    │   ├── store/droneStore.js
    │   ├── hooks/
    │   │   ├── useWebSocket.js
    │   │   └── useKeyboardShortcuts.js
    │   └── components/
    │       ├── layout/       # TopBar, FleetSidebar, RightPanel
    │       ├── map/          # MapView, DroneMarker, MissionOverlay
    │       ├── panels/       # Telemetry, Command, Mission, Log
    │       └── ui/           # AlertBanner, ConfirmModal, ArtificialHorizon
    ├── index.html
    └── vite.config.js
```

---

## Getting Started

### Prerequisites
- Python 3.10+
- Node.js 18+
- ArduPilot SITL (for simulation) or real drones on UDP

### Backend

```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

### Frontend

```bash
cd frontend
npm install
npm run dev
# Open http://localhost:5173
```

### Simulate Drones (ArduPilot SITL)

```bash
sim_vehicle.py -v ArduCopter -I0 --out=udp:127.0.0.1:14550 --no-mavproxy
sim_vehicle.py -v ArduCopter -I1 --out=udp:127.0.0.1:14551 --no-mavproxy
sim_vehicle.py -v ArduCopter -I2 --out=udp:127.0.0.1:14552 --no-mavproxy
```

The backend listens on ports `14550`–`14552` by default. Adjust in `mavlink_bridge.py` to match your swarm size.

---

## MAVLink Command Reference

| Command | MAVLink Message | Notes |
|---|---|---|
| ARM | `MAV_CMD_COMPONENT_ARM_DISARM` p1=1 | Requires GPS fix |
| DISARM | `MAV_CMD_COMPONENT_ARM_DISARM` p1=0 | |
| TAKEOFF | `MAV_CMD_NAV_TAKEOFF` p7=alt | Altitude in metres |
| LAND | `MAV_CMD_NAV_LAND` | |
| RTL | `MAV_CMD_NAV_RETURN_TO_LAUNCH` | |
| HOLD | `MAV_CMD_DO_PAUSE_CONTINUE` p1=0 | Pause mission |
| RESUME | `MAV_CMD_DO_PAUSE_CONTINUE` p1=1 | |

---

## License

MIT
