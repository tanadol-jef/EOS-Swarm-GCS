# Swarm Drone Ground Station — Build Plan v2
> Web Application · MAVLink v2 · React + FastAPI · Operator-First UI · RTSP Video

---

## Progress

| # | Task | Status |
|---|---|---|
| — | Write build plan (gs.md v2) | ✅ Done |
| — | Write README.md | ✅ Done |
| — | Create GitHub repo (EOS-Swarm-GCS) | ✅ Done — https://github.com/tanadol-jef/EOS-Swarm-GCS |
| — | Add RTSP video feed to plan (Phase 8) | ✅ Done |
| 1 | Scaffold dirs, init Vite, install deps | ⬜ Not started |
| 2 | `droneStore.js` + `useWebSocket.js` | ⬜ Not started |
| 3 | `main.py` + `mavlink_bridge.py` + `drone_registry.py` | ⬜ Not started |
| 4 | `App.jsx` shell + dark CSS | ⬜ Not started |
| 5 | `TopBar` + `FleetSidebar` | ⬜ Not started |
| 6 | `MapView` + `DroneMarker` | ⬜ Not started |
| 7 | `TelemetryPanel` + `ArtificialHorizon` | ⬜ Not started |
| 8 | `CommandPanel` + `ConfirmModal` | ⬜ Not started |
| 9 | `AlertBanner` + auto-alert logic | ⬜ Not started |
| 10 | `MissionPanel` + waypoint upload | ⬜ Not started |
| 11 | Keyboard shortcuts | ⬜ Not started |
| 12 | Formation presets + geofence overlay | ⬜ Not started |
| 13 | `video_manager.py` + video REST endpoints | ⬜ Not started |
| 14 | `VideoFeedPanel.jsx` + `hls.js` | ⬜ Not started |

---

## Project Overview

A browser-based Ground Control Station (GCS) for managing a swarm of drones. Designed for real field use: dark theme, keyboard shortcuts, instant status-at-a-glance, and safety confirmations on destructive commands.

---

## Architecture

```
[Drone 1] ──UDP:14550──┐
[Drone 2] ──UDP:14551──┤──► [FastAPI + pymavlink Bridge] ──WS──► [React App]
[Drone N] ──UDP:14552──┘         (one thread per drone)              (Vite + Zustand)
```

---

## Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Frontend | React 18 + Vite | Fast HMR, small bundle |
| State | Zustand | Simple, no boilerplate |
| Map | Leaflet + react-leaflet | Mature, offline-capable |
| Styling | Tailwind CSS | Dark theme utility classes, fast iteration |
| Charts | recharts | Lightweight sparklines for telemetry trends |
| Video | hls.js | Browser HLS playback (RTSP proxy output) |
| Backend | Python FastAPI + uvicorn | Async-native, easy WebSocket |
| MAVLink | pymavlink | Official, well-tested |
| Video proxy | FFmpeg (system binary) | RTSP → HLS transcoding per drone |
| Icons | lucide-react | Clean, consistent icon set |

---

## Directory Structure

```
GGS/
├── backend/
│   ├── main.py               # FastAPI app, WS + video REST endpoints
│   ├── mavlink_bridge.py     # Per-drone UDP listeners + broadcast
│   ├── drone_registry.py     # Telemetry state + heartbeat timeout + rtsp_url
│   ├── command_handler.py    # MAVLink command dispatcher
│   ├── video_manager.py      # FFmpeg RTSP→HLS process manager
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── main.jsx
│   │   ├── App.jsx           # Root layout
│   │   ├── store/
│   │   │   └── droneStore.js
│   │   ├── hooks/
│   │   │   ├── useWebSocket.js
│   │   │   └── useKeyboardShortcuts.js
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   │   ├── TopBar.jsx          # Connection status + swarm health
│   │   │   │   ├── FleetSidebar.jsx    # Drone list, left panel
│   │   │   │   └── RightPanel.jsx      # Tab container: Telemetry / Log / Mission
│   │   │   ├── map/
│   │   │   │   ├── MapView.jsx         # Leaflet map, full-center
│   │   │   │   ├── DroneMarker.jsx     # SVG blip + heading arrow
│   │   │   │   └── MissionOverlay.jsx  # Waypoints + geofence polygon
│   │   │   ├── panels/
│   │   │   │   ├── TelemetryPanel.jsx  # Selected drone HUD data
│   │   │   │   ├── CommandPanel.jsx    # Per-drone + swarm commands
│   │   │   │   ├── MissionPanel.jsx    # Waypoint list, upload controls
│   │   │   │   ├── LogPanel.jsx        # MAVLink message stream
│   │   │   │   └── VideoFeedPanel.jsx  # RTSP stream selector + HLS player
│   │   │   └── ui/
│   │   │       ├── AlertBanner.jsx     # Top alert strip (WARNING/CRITICAL)
│   │   │       ├── ConfirmModal.jsx    # Safety confirmation dialog
│   │   │       ├── BatteryBar.jsx
│   │   │       ├── SignalDots.jsx
│   │   │       ├── ArtificialHorizon.jsx  # SVG attitude indicator
│   │   │       └── StatusBadge.jsx     # ARMED / DISARMED / LOST / RTL
│   │   └── styles/
│   │       └── globals.css
│   ├── index.html
│   ├── vite.config.js
│   └── package.json
└── gs.md
```

---

## UI Layout

```
┌─────────────────────────────────────────────────────────────────┐
│ TOP BAR: [● WS Connected]  Swarm: 3/3 OK  [⚠ Drone 2 Low Batt] │
├────────────┬────────────────────────────────┬────────────────────┤
│            │                                │  [Telemetry]       │
│ FLEET      │                                │  [Commands]        │
│ SIDEBAR    │        MAP (Leaflet)            │  [Mission]         │
│            │                                │  [Log]             │
│            │                                │  [Video]           │
│ Drone 1 ●  │   ✈ ✈ ✈ (blips w/ arrows)     │                    │
│ Drone 2 ⚠  │                                │  ← right panel     │
│ Drone 3 ●  │   [click drone → select]       │    switches tabs   │
│            │                                │    on drone select │
│ [ARM ALL]  │                                │                    │
│ [LAND ALL] │                                │                    │
│ [RTL ALL]  │                                │                    │
│ [E-STOP]   │                                │                    │
└────────────┴────────────────────────────────┴────────────────────┘
```

**Key UX decisions:**
- Map is always center — never hidden behind tabs
- Right panel context switches when you select a drone
- Alert banner appears only when there's an active warning (no banner = clear)
- Dark theme throughout (`bg-gray-950` base)
- E-STOP is red, always visible, one click + confirm modal

---

## Phase 1 — Backend: MAVLink Bridge

### `backend/requirements.txt`
```
fastapi
uvicorn[standard]
pymavlink
websockets
```

### `backend/drone_registry.py`
```python
import time

HEARTBEAT_TIMEOUT = 3.0  # seconds

class DroneRegistry:
    def __init__(self):
        self.drones = {}  # sysid -> state dict

    def update(self, sysid, msg_type, data):
        if sysid not in self.drones:
            self.drones[sysid] = {"sysid": sysid, "status": "IDLE", "rtsp_url": None}
        self.drones[sysid][msg_type] = data
        if msg_type == "HEARTBEAT":
            self.drones[sysid]["last_seen"] = time.time()
            self.drones[sysid]["status"] = self._parse_status(data)
        # Auto-populate RTSP URL from VIDEO_STREAM_INFORMATION if present
        if msg_type == "VIDEO_STREAM_INFORMATION":
            uri = data.get("uri", "").strip()
            if uri:
                self.drones[sysid]["rtsp_url"] = uri

    def check_timeouts(self):
        now = time.time()
        for sysid, drone in self.drones.items():
            if now - drone.get("last_seen", now) > HEARTBEAT_TIMEOUT:
                drone["status"] = "LOST"

    def _parse_status(self, hb):
        base_mode = hb.get("base_mode", 0)
        armed = bool(base_mode & 128)
        return "ARMED" if armed else "IDLE"

    def snapshot(self):
        # Omit large nested dicts (raw MAVLink messages) from snapshot;
        # keep only top-level scalar fields the frontend needs
        keys = {"sysid", "status", "rtsp_url", "last_seen"}
        return [{k: v for k, v in d.items() if k in keys} for d in self.drones.values()]
```

### `backend/mavlink_bridge.py`
```python
from pymavlink import mavutil
import asyncio, json, threading, time
from drone_registry import DroneRegistry

WATCHED_MSGS = {
    "HEARTBEAT", "GLOBAL_POSITION_INT", "SYS_STATUS",
    "ATTITUDE", "VFR_HUD", "GPS_RAW_INT",
    "MISSION_CURRENT", "STATUSTEXT", "NAV_CONTROLLER_OUTPUT",
    "VIDEO_STREAM_INFORMATION",  # Auterion drones broadcast this; contains RTSP URI
}

class MAVLinkBridge:
    def __init__(self, ports=(14550, 14551, 14552)):
        self.registry = DroneRegistry()
        self.clients = set()
        self.connections = {}  # sysid -> mavlink connection
        self.loop = asyncio.get_event_loop()
        for port in ports:
            threading.Thread(target=self._listen, args=(port,), daemon=True).start()
        threading.Thread(target=self._heartbeat_watchdog, daemon=True).start()

    def _listen(self, port):
        mav = mavutil.mavlink_connection(f"udpin:0.0.0.0:{port}")
        seen = set()
        while True:
            msg = mav.recv_match(blocking=True, timeout=1)
            if not msg:
                continue
            sysid = msg.get_srcSystem()
            if sysid not in self.connections:
                self.connections[sysid] = mav
            # On first contact with a drone, request its video stream info
            # MAV_CMD_REQUEST_MESSAGE (512) with param1 = 269 (VIDEO_STREAM_INFORMATION)
            if sysid not in seen:
                seen.add(sysid)
                mav.mav.command_long_send(
                    mav.target_system, mav.target_component,
                    mavutil.mavlink.MAV_CMD_REQUEST_MESSAGE,
                    0, 269, 0, 0, 0, 0, 0, 0,
                )
            if msg.get_type() in WATCHED_MSGS:
                self.registry.update(sysid, msg.get_type(), msg.to_dict())
                payload = {"type": msg.get_type(), "sysid": sysid, "data": msg.to_dict()}
                asyncio.run_coroutine_threadsafe(self._broadcast(payload), self.loop)

    def _heartbeat_watchdog(self):
        while True:
            time.sleep(1)
            self.registry.check_timeouts()
            payload = {"type": "REGISTRY_SNAPSHOT", "data": self.registry.snapshot()}
            asyncio.run_coroutine_threadsafe(self._broadcast(payload), self.loop)

    async def _broadcast(self, payload):
        dead = set()
        for ws in self.clients:
            try:
                await ws.send_text(json.dumps(payload))
            except Exception:
                dead.add(ws)
        self.clients -= dead

    def add_client(self, ws): self.clients.add(ws)
    def remove_client(self, ws): self.clients.discard(ws)

    async def handle_command(self, cmd):
        sysid = cmd.get("drone_id")
        command = cmd.get("command")
        params = cmd.get("params", {})
        mav = self.connections.get(sysid)
        if not mav:
            return
        from command_handler import dispatch
        dispatch(mav, command, params)
```

### `backend/command_handler.py`
```python
from pymavlink import mavutil

def dispatch(mav, command, params):
    handlers = {
        "ARM":     lambda: arm(mav, True),
        "DISARM":  lambda: arm(mav, False),
        "TAKEOFF": lambda: takeoff(mav, params.get("altitude", 10)),
        "LAND":    lambda: land(mav),
        "RTL":     lambda: rtl(mav),
        "HOLD":    lambda: hold(mav),
    }
    fn = handlers.get(command)
    if fn:
        fn()

def arm(mav, arm_state):
    mav.mav.command_long_send(
        mav.target_system, mav.target_component,
        mavutil.mavlink.MAV_CMD_COMPONENT_ARM_DISARM,
        0, 1 if arm_state else 0, 0, 0, 0, 0, 0, 0
    )

def takeoff(mav, altitude):
    mav.mav.command_long_send(
        mav.target_system, mav.target_component,
        mavutil.mavlink.MAV_CMD_NAV_TAKEOFF,
        0, 0, 0, 0, 0, 0, 0, altitude
    )

def land(mav):
    mav.mav.command_long_send(
        mav.target_system, mav.target_component,
        mavutil.mavlink.MAV_CMD_NAV_LAND,
        0, 0, 0, 0, 0, 0, 0, 0
    )

def rtl(mav):
    mav.mav.command_long_send(
        mav.target_system, mav.target_component,
        mavutil.mavlink.MAV_CMD_NAV_RETURN_TO_LAUNCH,
        0, 0, 0, 0, 0, 0, 0, 0
    )

def hold(mav):
    mav.mav.command_long_send(
        mav.target_system, mav.target_component,
        mavutil.mavlink.MAV_CMD_DO_PAUSE_CONTINUE,
        0, 0, 0, 0, 0, 0, 0, 0
    )
```

### `backend/main.py`
```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import FileResponse, JSONResponse
from mavlink_bridge import MAVLinkBridge
from video_manager import VideoManager, STREAMS_DIR
import json

app = FastAPI()
app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_methods=["*"], allow_headers=["*"])

bridge = MAVLinkBridge(ports=[14550, 14551, 14552])
video  = VideoManager()

# ── WebSocket ────────────────────────────────────────────────────────────────

@app.websocket("/ws")
async def ws_endpoint(websocket: WebSocket):
    await websocket.accept()
    bridge.add_client(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            cmd = json.loads(data)
            await bridge.handle_command(cmd)
    except WebSocketDisconnect:
        bridge.remove_client(websocket)

# ── Video REST ───────────────────────────────────────────────────────────────

@app.post("/stream/{sysid}/start")
async def start_stream(sysid: int, body: dict):
    rtsp_url = body.get("rtsp_url", "").strip()
    if not rtsp_url:
        return JSONResponse({"error": "rtsp_url required"}, status_code=400)
    hls_path = video.start_stream(sysid, rtsp_url)
    return {"status": "started", "hls_url": hls_path}

@app.delete("/stream/{sysid}/stop")
async def stop_stream(sysid: int):
    video.stop_stream(sysid)
    return {"status": "stopped"}

@app.get("/stream/{sysid}/status")
async def stream_status(sysid: int):
    return {"status": video.get_status(sysid)}

# Serve HLS playlist + segments
@app.get("/stream/{sysid}/hls/{filename}")
async def serve_hls(sysid: int, filename: str):
    path = video.stream_dir(sysid) / filename
    if not path.exists():
        return JSONResponse({"error": "not found"}, status_code=404)
    mime = "application/vnd.apple.mpegurl" if filename.endswith(".m3u8") else "video/mp2t"
    return FileResponse(str(path), media_type=mime)

@app.on_event("shutdown")
def on_shutdown():
    video.stop_all()
```

---

## Phase 2 — Frontend: State & WebSocket

### `src/store/droneStore.js`
```javascript
import { create } from "zustand";

export const useDroneStore = create((set, get) => ({
  connected: false,
  drones: {},        // sysid -> full telemetry
  selectedId: null,  // sysid of selected drone
  logs: [],
  alerts: [],        // { id, sysid, level, text }

  setConnected: (v) => set({ connected: v }),
  selectDrone: (id) => set({ selectedId: id }),

  applySnapshot: (snapshot) => {
    const map = {};
    snapshot.forEach((d) => { map[d.sysid] = { ...get().drones[d.sysid], ...d }; });
    set({ drones: map });
  },

  updateDrone: (sysid, msgType, data) =>
    set((s) => ({
      drones: {
        ...s.drones,
        [sysid]: { ...s.drones[sysid], sysid, [msgType]: data },
      },
    })),

  addLog: (msg) =>
    set((s) => ({
      logs: [{ ...msg, ts: new Date().toISOString() }, ...s.logs].slice(0, 500),
    })),

  pushAlert: (alert) =>
    set((s) => ({
      alerts: [{ id: Date.now(), ...alert }, ...s.alerts].slice(0, 20),
    })),

  dismissAlert: (id) =>
    set((s) => ({ alerts: s.alerts.filter((a) => a.id !== id) })),
}));
```

### `src/hooks/useWebSocket.js`
```javascript
import { useEffect, useRef, useCallback } from "react";
import { useDroneStore } from "../store/droneStore";

const RECONNECT_DELAY = 2000;

export function useWebSocket(url = "ws://localhost:8000/ws") {
  const ws = useRef(null);
  const retryTimer = useRef(null);
  const { setConnected, updateDrone, applySnapshot, addLog, pushAlert } = useDroneStore();

  const connect = useCallback(() => {
    ws.current = new WebSocket(url);

    ws.current.onopen = () => setConnected(true);

    ws.current.onclose = () => {
      setConnected(false);
      retryTimer.current = setTimeout(connect, RECONNECT_DELAY);
    };

    ws.current.onmessage = (e) => {
      const msg = JSON.parse(e.data);
      if (msg.type === "REGISTRY_SNAPSHOT") {
        applySnapshot(msg.data);
        return;
      }
      updateDrone(msg.sysid, msg.type, msg.data);
      addLog(msg);
      // Auto-alerts
      if (msg.type === "SYS_STATUS") {
        const batt = msg.data.battery_remaining;
        if (batt < 20 && batt >= 0) {
          pushAlert({ sysid: msg.sysid, level: "WARNING", text: `Drone ${msg.sysid} battery ${batt}%` });
        }
      }
      if (msg.type === "STATUSTEXT" && msg.data.severity <= 3) {
        pushAlert({ sysid: msg.sysid, level: "CRITICAL", text: msg.data.text });
      }
    };
  }, [url]);

  useEffect(() => {
    connect();
    return () => {
      clearTimeout(retryTimer.current);
      ws.current?.close();
    };
  }, [connect]);

  const sendCommand = useCallback((droneId, command, params = {}) => {
    if (ws.current?.readyState === WebSocket.OPEN) {
      ws.current.send(JSON.stringify({ drone_id: droneId, command, params }));
    }
  }, []);

  const sendSwarmCommand = useCallback((command, params = {}) => {
    const { drones } = useDroneStore.getState();
    Object.keys(drones).forEach((id) => sendCommand(Number(id), command, params));
  }, [sendCommand]);

  return { sendCommand, sendSwarmCommand };
}
```

### `src/hooks/useKeyboardShortcuts.js`
```javascript
import { useEffect } from "react";

export function useKeyboardShortcuts({ onEmergencyStop, onLandAll, onRTLAll }) {
  useEffect(() => {
    const handler = (e) => {
      if (e.target.tagName === "INPUT") return;
      if (e.key === "Escape") onEmergencyStop?.();
      if (e.key === "l" && e.ctrlKey) onLandAll?.();
      if (e.key === "r" && e.ctrlKey) onRTLAll?.();
    };
    window.addEventListener("keydown", handler);
    return () => window.removeEventListener("keydown", handler);
  }, [onEmergencyStop, onLandAll, onRTLAll]);
}
```

---

## Phase 3 — UI Components

### `src/App.jsx` (root layout)
```jsx
import { useWebSocket } from "./hooks/useWebSocket";
import { useDroneStore } from "./store/droneStore";
import TopBar from "./components/layout/TopBar";
import FleetSidebar from "./components/layout/FleetSidebar";
import MapView from "./components/map/MapView";
import RightPanel from "./components/layout/RightPanel";
import AlertBanner from "./components/ui/AlertBanner";

export default function App() {
  const { sendCommand, sendSwarmCommand } = useWebSocket();

  return (
    <div className="flex flex-col h-screen bg-gray-950 text-gray-100 font-mono overflow-hidden">
      <TopBar />
      <AlertBanner />
      <div className="flex flex-1 overflow-hidden">
        <FleetSidebar sendSwarmCommand={sendSwarmCommand} />
        <MapView />
        <RightPanel sendCommand={sendCommand} />
      </div>
    </div>
  );
}
```

### TopBar — connection + swarm health
```jsx
// Shows: WS dot, drone count, active alerts count
// Always visible — operator knows at a glance if backend is alive
```

### FleetSidebar — drone list + swarm buttons
```jsx
// Each drone card shows:
//   [Status badge] Drone N
//   [Battery bar] [Signal dots]
//   Last seen timestamp
// Click card → selectDrone(sysid)
// Bottom section: ARM ALL / LAND ALL / RTL ALL / [E-STOP]
// E-STOP opens ConfirmModal before sending
```

### MapView — Leaflet, always visible
```jsx
// Dark tile layer: CartoDB Dark Matter or offline tiles
// DroneMarker per drone: color = status (green/yellow/red/gray)
// Heading arrow rotates with ATTITUDE.yaw
// Click marker → selectDrone
// MissionOverlay: polyline + numbered waypoint markers
// Geofence polygon with transparent fill
```

### RightPanel — context tabs
```jsx
// Tabs: Telemetry | Commands | Mission | Log | Video
// Auto-switches to Telemetry when drone selected
// TelemetryPanel: lat/lon/alt, speed, heading, battery%, mode, armed, GPS fix
//   + ArtificialHorizon SVG (roll/pitch from ATTITUDE)
//   + altitude sparkline (recharts, last 60s)
// CommandPanel: ARM / DISARM / TAKEOFF (altitude input) / LAND / RTL / HOLD
//   Dangerous commands (ARM, DISARM) require confirm modal
// MissionPanel: waypoint list, [Upload] [Clear] buttons, formation presets
// LogPanel: scrollable MAVLink stream, filter by sysid or type
// VideoFeedPanel: drone selector + RTSP URL input + HLS player (see Phase 8)
```

### ConfirmModal — safety dialog
```jsx
// Props: message, onConfirm, onCancel
// Shown for: ARM, DISARM, ARM ALL, DISARM ALL, E-STOP
// E-STOP modal has red background, large text
// Auto-focus confirm button; Enter = confirm, Escape = cancel
```

---

## Phase 4 — MAVLink Commands Reference

| UI Button | MAVLink | Notes |
|---|---|---|
| ARM | `MAV_CMD_COMPONENT_ARM_DISARM` p1=1 | Requires GPS fix + pre-arm OK |
| DISARM | `MAV_CMD_COMPONENT_ARM_DISARM` p1=0 | Confirm dialog |
| TAKEOFF | `MAV_CMD_NAV_TAKEOFF` p7=alt | Altitude input (m) |
| LAND | `MAV_CMD_NAV_LAND` | |
| RTL | `MAV_CMD_NAV_RETURN_TO_LAUNCH` | |
| HOLD | `MAV_CMD_DO_PAUSE_CONTINUE` p1=0 | Pause mission |
| RESUME | `MAV_CMD_DO_PAUSE_CONTINUE` p1=1 | |
| SET_MODE | `MAV_CMD_DO_SET_MODE` | |

---

## Phase 5 — Mission Planning

### Waypoint Upload Flow
1. User clicks map in mission edit mode → waypoints array appended
2. Waypoints listed in MissionPanel (drag to reorder)
3. [Upload Mission] → JSON → WebSocket → backend serializes to `MISSION_ITEM_INT`
4. Backend sends `MISSION_COUNT` then streams items; drone replies `MISSION_ACK`
5. Progress shown in MissionPanel

### Formation Presets
```javascript
// Grid — evenly spaced rows/cols
export function gridFormation(center, count, spacing = 10) {
  const cols = Math.ceil(Math.sqrt(count));
  return Array.from({ length: count }, (_, i) => ({
    lat: center.lat + Math.floor(i / cols) * (spacing / 111111),
    lng: center.lng + (i % cols) * (spacing / (111111 * Math.cos(center.lat * Math.PI / 180))),
  }));
}

// Line — single row
export function lineFormation(center, count, spacing = 10, bearing = 0) {
  const rad = bearing * Math.PI / 180;
  return Array.from({ length: count }, (_, i) => ({
    lat: center.lat + (i - (count - 1) / 2) * (spacing / 111111) * Math.cos(rad),
    lng: center.lng + (i - (count - 1) / 2) * (spacing / 111111) * Math.sin(rad),
  }));
}
```

---

## Phase 6 — Safety System

| Condition | Trigger | Action |
|---|---|---|
| No HEARTBEAT for 3s | Watchdog thread | Status → LOST, alert banner |
| Battery < 20% | SYS_STATUS | Warning alert, badge flashes |
| Battery < 10% | SYS_STATUS | Auto-RTL (configurable) |
| STATUSTEXT severity ≤ 3 | Message | Critical alert banner |
| Geofence breach | Frontend position check | Warning alert |
| E-STOP pressed | User | Disarm ALL with confirm modal |

---

## Phase 7 — Styling: Dark Theme Tokens

```css
/* globals.css */
:root {
  --bg-base:     #030712;   /* gray-950 */
  --bg-panel:    #111827;   /* gray-900 */
  --bg-card:     #1f2937;   /* gray-800 */
  --border:      #374151;   /* gray-700 */
  --text-main:   #f9fafb;   /* gray-50  */
  --text-muted:  #9ca3af;   /* gray-400 */

  --status-ok:   #22c55e;   /* green-500 */
  --status-warn: #f59e0b;   /* amber-500 */
  --status-crit: #ef4444;   /* red-500  */
  --status-lost: #6b7280;   /* gray-500 */
  --status-rtl:  #a855f7;   /* purple-500 */
}
```

---

## Keyboard Shortcuts

| Key | Action |
|---|---|
| `Esc` | Emergency stop (opens confirm) |
| `Ctrl+L` | Land all |
| `Ctrl+R` | RTL all |
| `1`–`9` | Select drone by index |
| `Tab` | Cycle right panel tabs |

---

## Phase 8 — Video Feed (RTSP / HLS)

### Overview

Auterion OS drones expose camera video via RTSP. Browsers cannot consume RTSP directly, so the backend uses FFmpeg to pull each drone's RTSP stream and re-serve it as HLS. The frontend plays HLS using `hls.js`.

```
[Auterion Drone]
  RTSP :8554/live
       │
       ▼  (FFmpeg, one process per active stream)
[Backend /stream/{sysid}/hls/]
  index.m3u8 + seg*.ts  (served as static files)
       │
       ▼  (hls.js, HTTP GET)
[VideoFeedPanel <video> element]
```

**Latency:** ~2–4 s with 1 s HLS segments (acceptable for situational awareness).  
**CPU:** FFmpeg uses `-c:v copy` (pass-through, no re-encode) — minimal load.

---

### RTSP URL Auto-Detection

When the backend first sees a drone, it sends `MAV_CMD_REQUEST_MESSAGE` (param1 = 269) to request `VIDEO_STREAM_INFORMATION`. Auterion drones respond with the full RTSP URI in the `uri` field. This URI is stored in `DroneRegistry` as `rtsp_url` and included in the `REGISTRY_SNAPSHOT` broadcast so the frontend can pre-fill the URL input.

**Default Auterion / Skynode RTSP patterns (manual fallback):**
| Camera | URL |
|---|---|
| Main payload camera | `rtsp://<drone-ip>:8554/live` |
| Thermal (if equipped) | `rtsp://<drone-ip>:8554/thermal` |
| Auterion Suite proxy | `rtsp://<suite-ip>/drone-<sysid>` |

---

### `backend/video_manager.py`

```python
import subprocess, threading
from pathlib import Path
import tempfile

STREAMS_DIR = Path(tempfile.gettempdir()) / "gcs_streams"
HLS_TIME      = "1"   # seconds per segment — lower = less latency, higher CPU
HLS_LIST_SIZE = "3"   # segments kept in playlist

class VideoManager:
    def __init__(self):
        self._procs: dict[int, subprocess.Popen] = {}
        self._lock = threading.Lock()
        STREAMS_DIR.mkdir(exist_ok=True)

    def start_stream(self, sysid: int, rtsp_url: str) -> str:
        """Kill any existing stream, start FFmpeg, return HLS URL path."""
        self.stop_stream(sysid)
        out_dir = STREAMS_DIR / str(sysid)
        out_dir.mkdir(exist_ok=True)
        # Clear stale segments so hls.js doesn't pick up old data
        for f in out_dir.glob("*.ts"):   f.unlink(missing_ok=True)
        for f in out_dir.glob("*.m3u8"): f.unlink(missing_ok=True)

        cmd = [
            "ffmpeg", "-y",
            "-rtsp_transport", "tcp",          # TCP avoids UDP packet loss
            "-i", rtsp_url,
            "-c:v", "copy",                    # pass-through — no re-encode
            "-an",                             # drop audio
            "-f", "hls",
            "-hls_time", HLS_TIME,
            "-hls_list_size", HLS_LIST_SIZE,
            "-hls_flags", "delete_segments+omit_endlist+append_list",
            "-hls_segment_type", "mpegts",
            "-hls_segment_filename", str(out_dir / "seg%03d.ts"),
            str(out_dir / "index.m3u8"),
        ]
        proc = subprocess.Popen(cmd, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
        with self._lock:
            self._procs[sysid] = proc
        return f"/stream/{sysid}/hls/index.m3u8"

    def stop_stream(self, sysid: int):
        with self._lock:
            proc = self._procs.pop(sysid, None)
        if proc and proc.poll() is None:
            proc.terminate()
            try: proc.wait(timeout=3)
            except subprocess.TimeoutExpired: proc.kill()

    def stop_all(self):
        for sysid in list(self._procs): self.stop_stream(sysid)

    def get_status(self, sysid: int) -> str:
        with self._lock:
            proc = self._procs.get(sysid)
        if not proc:       return "stopped"
        if proc.poll() is None: return "running"
        return "error"

    def stream_dir(self, sysid: int) -> Path:
        return STREAMS_DIR / str(sysid)
```

---

### `frontend/src/components/panels/VideoFeedPanel.jsx`

**Component state machine:**
```
stopped ──[Connect]──► connecting ──[HLS parsed]──► live
   ▲                        │                         │
   └──────[Disconnect]──────┘         [HLS error]─────┘
                                           ▼
                                         error ──[Retry]──► connecting
```

**Layout:**

```
┌─────────────────────────────────────┐
│ Drone: [Drone 2 ▾]  ● LIVE          │  ← selector + status dot
│ rtsp://10.41.1.1:8554/live    [✎]   │  ← auto-filled, editable
│ [▶ Connect]        [■ Disconnect]   │  ← action buttons
├─────────────────────────────────────┤
│                                     │
│          <video> (16:9)             │  ← hls.js player
│                             [⛶]    │  ← fullscreen button
└─────────────────────────────────────┘
```

**Key behaviour:**
- On drone select → auto-fill `rtsp_url` from `drone.rtsp_url` if present (from `VIDEO_STREAM_INFORMATION`)
- If no `rtsp_url` known → show placeholder `rtsp://<drone-ip>:8554/live`, keep editable
- **Connect:** `POST /stream/{sysid}/start { rtsp_url }` → start HLS via `hls.js`
- **Disconnect:** `DELETE /stream/{sysid}/stop` + destroy `hls.js` instance
- Poll `GET /stream/{sysid}/status` every 3 s while live → surface FFmpeg crashes
- Auto-retry on HLS fatal error (3 attempts, then show error state)
- Switching drone selection disconnects existing stream first
- Fullscreen: `videoElement.requestFullscreen()`
- `<video>` is always in the DOM; hidden when not live (avoids layout shift on connect)

**npm dep to add:** `hls.js`
```bash
npm install hls.js
```

**hls.js config for low latency:**
```javascript
const hls = new Hls({
  lowLatencyMode: true,
  liveSyncDurationCount: 2,
  liveMaxLatencyDurationCount: 4,
});
```

---

### Zustand — video slice additions to `droneStore.js`

```javascript
// Add to store:
activeStream: null,   // { sysid, status: "connecting"|"live"|"error" }
setStreamStatus: (sysid, status) => set({ activeStream: { sysid, status } }),
clearStream: () => set({ activeStream: null }),
```

The `REGISTRY_SNAPSHOT` handler already writes `drone.rtsp_url` from the snapshot — no extra action needed on the store side.

---

### Requirements update

```
# backend/requirements.txt — add nothing; FFmpeg must be installed as a system binary
# frontend — add:
npm install hls.js
```

FFmpeg install (per environment):
```bash
# Ubuntu / Debian
sudo apt install ffmpeg

# macOS
brew install ffmpeg

# Windows — download from https://ffmpeg.org/download.html, add to PATH
```

---

## Running the Stack

```bash
# Terminal 1 — Backend
cd GGS/backend
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000 --reload

# Terminal 2 — Frontend
cd GGS/frontend
npm install
npm run dev
# → http://localhost:5173

# Terminal 3 — SITL (ArduCopter, 3 drones)
sim_vehicle.py -v ArduCopter -I0 --out=udp:127.0.0.1:14550 --no-mavproxy
sim_vehicle.py -v ArduCopter -I1 --out=udp:127.0.0.1:14551 --no-mavproxy
sim_vehicle.py -v ArduCopter -I2 --out=udp:127.0.0.1:14552 --no-mavproxy
```

---

## Build Order (Step by Step)

| Step | Task | Output |
|---|---|---|
| 1 | Scaffold dirs, init Vite, install deps | Empty app runs |
| 2 | `droneStore.js` + `useWebSocket.js` | State layer done |
| 3 | `main.py` + `mavlink_bridge.py` + `drone_registry.py` | Backend connects to SITL |
| 4 | `App.jsx` shell + dark CSS | Layout visible |
| 5 | `TopBar` + `FleetSidebar` (static data) | Sidebar renders |
| 6 | `MapView` + `DroneMarker` | Blips on map |
| 7 | `TelemetryPanel` + `ArtificialHorizon` | Live data panel |
| 8 | `CommandPanel` + `ConfirmModal` | Commands fire |
| 9 | `AlertBanner` + auto-alert logic | Safety feedback |
| 10 | `MissionPanel` + waypoint upload | Mission planning |
| 11 | Keyboard shortcuts | Power-user ops |
| 12 | Formation presets + geofence overlay | Swarm ops |
| 13 | `video_manager.py` + video REST endpoints in `main.py` | Backend can proxy RTSP |
| 14 | `VideoFeedPanel.jsx` + `hls.js` + video slice in store | Live drone video in browser |
