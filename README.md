Save as src/Dashboard.jsx

jsx
Copy
Edit
import React, { useState, useEffect, useRef } from "react";
import "./Dashboard.css";

/**
 * Dashboard.jsx
 * - live clock (updates every second)
 * - random realistic stats (updates every 3s)
 * - smooth transitions/animations
 *
 * Change ranges or update frequency in the useEffect below if needed.
 */

export default function Dashboard() {
  // Clock
  const [now, setNow] = useState(new Date());

  // Live stats
  const [battery, setBattery] = useState(86);
  const [suction, setSuction] = useState(2200); // Pa
  const [area, setArea] = useState(32); // m²
  const [runTime, setRunTime] = useState("00:45"); // mm:ss
  const [speed, setSpeed] = useState(0.8); // m/s
  const [mode, setMode] = useState("Auto");

  // For gentle number animation (interpolate)
  const animRef = useRef({ battery, suction, area, speed });

  // update clock every second
  useEffect(() => {
    const t = setInterval(() => setNow(new Date()), 1000);
    return () => clearInterval(t);
  }, []);

  // random-ish live updates every 3 seconds
  useEffect(() => {
    const up = () => {
      setBattery((b) => clamp(b + randInt(-3, 2), 0, 100));
      setSuction((s) => clamp(s + randInt(-150, 150), 500, 3000));
      setArea((a) => clamp(a + randInt(0, 3), 0, 200));
      setSpeed((sp) => +(clamp(sp + randFloat(-0.2, 0.2), 0.1, 1.8)).toFixed(2));
      setRunTime((rt) => {
        // increment minutes by random 1-3 minutes to mimic running timer or keep format
        const [mm, ss] = rt.split(":").map(Number);
        let totalSec = mm * 60 + ss + randInt(30, 120);
        const newMm = String(Math.floor(totalSec / 60)).padStart(2, "0");
        const newSs = String(totalSec % 60).padStart(2, "0");
        return `${newMm}:${newSs}`;
      });
      // Occasionally toggle mode
      if (Math.random() < 0.15) setMode((m) => (m === "Auto" ? "Spot" : m === "Spot" ? "Edge" : "Auto"));
    };

    up(); // initial
    const id = setInterval(up, 3000);
    return () => clearInterval(id);
  }, []);

  // helper functions
  function randInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }
  function randFloat(min, max) {
    return Math.random() * (max - min) + min;
  }
  function clamp(v, lo, hi) {
    return Math.max(lo, Math.min(hi, v));
  }

  // for progress width (battery)
  const batteryPct = battery;

  // formatted date/time
  const dateStr = now.toLocaleDateString(undefined, { weekday: "short", day: "numeric", month: "short", year: "numeric" });
  const timeStr = now.toLocaleTimeString(undefined, { hour: "2-digit", minute: "2-digit", second: "2-digit" });

  return (
    <div className="rdash-root">
      <header className="rdash-header">
        <div className="brand">
          <div className="logo-dot" />
          <h1>Cleaner • Dashboard</h1>
        </div>
        <div className="header-right">
          <div className="datetime">
            <div className="date">{dateStr}</div>
            <div className="time">{timeStr}</div>
          </div>
        </div>
      </header>

      <main className="rdash-main">
        {/* Left: Big status card */}
        <section className="card big-card">
          <div className="big-top">
            <div className="big-title">Suction Power</div>
            <div className="big-value">{suction} Pa</div>
          </div>

          <div className="big-mid">
            <div className="gauge">
              <div className="gauge-needle" style={{ transform: `rotate(${map(suction, 500, 3000, -90, 90)}deg)` }} />
              <div className="gauge-center" />
            </div>

            <div className="big-stats">
              <div className="stat">
                <div className="stat-label">Mode</div>
                <div className="stat-value">{mode}</div>
              </div>
              <div className="stat">
                <div className="stat-label">Run Time</div>
                <div className="stat-value">{runTime}</div>
              </div>
              <div className="stat">
                <div className="stat-label">Speed</div>
                <div className="stat-value">{speed} m/s</div>
              </div>
            </div>
          </div>

          <div className="big-bottom">
            <div className="battery-wrap">
              <div className="battery-label">Battery</div>
              <div className="battery-bar">
                <div className="battery-fill" style={{ width: `${batteryPct}%` }} />
              </div>
              <div className="battery-pct">{batteryPct}%</div>
            </div>

            <div className="area-wrap">
              <div className="area-label">Cleaned</div>
              <div className="area-value">{area} m²</div>
            </div>

            <div className="controls">
              <button className="btn pause">Pause</button>
              <button className="btn dock">Dock</button>
            </div>
          </div>
        </section>

        {/* Middle column: small cards */}
        <section className="col mid-col">
          <div className="card small-card">
            <div className="small-title">Brush Status</div>
            <div className="small-body">
              <div className="circle-ind">OK</div>
              <div className="small-sub">Last cleaned 12m ago</div>
            </div>
          </div>

          <div className="card small-card">
            <div className="small-title">Filters</div>
            <div className="small-body">
              <div className="filter-bar">
                <div className="filter-fill" style={{ width: `${clamp(100 - dustLevelPlaceholder(0), 10, 100)}%` }} />
              </div>
              <div className="small-sub">Replace in 14 days</div>
            </div>
          </div>

          <div className="card small-card">
            <div className="small-title">Map</div>
            <div className="small-body map-box">— floor map —</div>
          </div>
        </section>

        {/* Right column: metrics */}
        <section className="col right-col">
          <div className="card metric-card">
            <div className="metric-title">Battery</div>
            <div className="metric-value">{battery}%</div>
            <div className="metric-bar">
              <div className="metric-fill" style={{ width: `${battery}%` }} />
            </div>
          </div>

          <div className="card metric-card">
            <div className="metric-title">Dust Level</div>
            <div className="metric-value">—</div>
            <div className="metric-sub">Sensor OK</div>
          </div>

          <div className="card metric-card">
            <div className="metric-title">Next Cleaning</div>
            <div className="metric-value">Tomorrow 09:00</div>
            <div className="metric-sub">Recurring: Weekly</div>
          </div>

          <div className="card metric-card">
            <div className="metric-title">Controls</div>
            <div className="controls-col">
              <button className="btn small">Start</button>
              <button className="btn small outline">Stop</button>
              <button className="btn small">Locate</button>
            </div>
          </div>
        </section>
      </main>
    </div>
  );
}

/* small helpers */
function map(x, inMin, inMax, outMin, outMax) {
  const t = (x - inMin) / (inMax - inMin);
  return outMin + t * (outMax - outMin);
}
// placeholder dust level function used only for visual filter bar (can link to real state)
function dustLevelPlaceholder(seed = 0) {
  return 25 + (seed % 30);
}
2) Dashboard.css
Save as src/Dashboard.css

css
Copy
Edit
/* Dashboard.css
   Matches layout from provided image: header, big left card, mid small cards, right metric column.
   Uses soft shadows, rounded cards and smooth transitions.
*/

:root{
  --bg: #f2f6fb;
  --card: #ffffff;
  --muted: #7b7f87;
  --accent: #2f69f6;
  --success: #28a745;
  --danger: #ff6b6b;
  --shadow: 0 8px 20px rgba(17,24,39,0.08);
}

*{box-sizing:border-box}
body,html,#root{height:100%}
.rdash-root{
  min-height:100vh;
  background: linear-gradient(180deg, var(--bg) 0%, #eef3fb 100%);
  padding: 22px;
  font-family: Inter, "Segoe UI", Roboto, Arial, sans-serif;
}

/* header */
.rdash-header{
  display:flex;
  justify-content:space-between;
  align-items:center;
  margin-bottom:18px;
}
.brand{display:flex;align-items:center;gap:12px}
.logo-dot{
  width:12px;height:12px;border-radius:50%;background:var(--accent);box-shadow:0 0 10px rgba(47,105,246,0.3)
}
.brand h1{font-size:18px;margin:0;color:#1f2937;font-weight:600}
.header-right{display:flex;align-items:center;gap:10px}
.datetime{ text-align:right; color:var(--muted); font-size:13px}
.datetime .date{font-weight:600;color:#111827}
.datetime .time{font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, "Roboto Mono", monospace}

/* main layout */
.rdash-main{
  display:flex;
  gap:20px;
  align-items:flex-start;
}

/* left big card */
.big-card{
  flex: 1.3;
  min-width: 420px;
  background:var(--card);
  padding:18px;
  border-radius:14px;
  box-shadow:var(--shadow);
  display:flex;
  flex-direction:column;
  justify-content:space-between;
  transition: transform .18s ease;
}
.big-card:hover{ transform: translateY(-4px) }

.big-top{ display:flex; justify-content:space-between; align-items:center; margin-bottom:12px }
.big-title{ color:#374151; font-weight:600; font-size:16px }
.big-value{ font-size:22px; color:var(--accent); font-weight:700 }

/* middle area */
.big-mid{ display:flex; gap:18px; align-items:center; margin:12px 0 }
.gauge{
  width:160px; height:120px; position:relative;
  background: linear-gradient(180deg,#fff,#f6fbff);
  border-radius:100px;
  display:flex; align-items:center; justify-content:center;
  box-shadow: 0 6px 18px rgba(47,105,246,0.06);
}
.gauge-needle{
  position:absolute;
  width:2px; height:46%;
  background: #ff6b6b;
  transform-origin: bottom center;
  transition: transform 700ms cubic-bezier(.22,.9,.2,1);
}
.gauge-center{
  width:14px; height:14px; border-radius:50%; background:#111827; z-index:2;
}

.big-stats{ display:flex; flex-direction:column; gap:8px; width:100% }
.stat{ display:flex; justify-content:space-between; align-items:center; padding:8px 10px; border-radius:10px; background:#fbfdff }
.stat-label{ color:var(--muted); font-size:13px }
.stat-value{ font-weight:700; color:#111827 }

/* bottom */
.big-bottom{ display:flex; gap:18px; align-items:center; margin-top:14px }
.battery-wrap{ flex:1 }
.battery-label{ color:var(--muted); font-size:13px; margin-bottom:6px }
.battery-bar{ width:100%; height:12px; background:#eef2ff; border-radius:12px; overflow:hidden; box-shadow: inset 0 1px 0 rgba(255,255,255,0.6) }
.battery-fill{ height:100%; background: linear-gradient(90deg,#26a9ff,#2f69f6); width:60%; transition: width 700ms ease; }

.battery-pct{ font-weight:700; color:#111827; margin-top:6px }

.area-wrap{ width:110px; text-align:center }
.area-label{ color:var(--muted); font-size:13px }
.area-value{ font-weight:700; font-size:18px; margin-top:6px }

.controls{ display:flex; gap:8px; align-items:center }
.btn{ padding:8px 12px; border-radius:8px; border:none; cursor:pointer; font-weight:600; }
.btn.pause{ background:#fff3cd; color:#856404; box-shadow: 0 6px 12px rgba(0,0,0,0.04) }
.btn.dock{ background:#2f69f6; color:white }

/* middle column small cards */
.col.mid-col{ width:280px; display:flex; flex-direction:column; gap:14px }
.card.small-card{ background:var(--card); padding:14px; border-radius:12px; box-shadow:var(--shadow); display:flex; flex-direction:column; gap:10px; min-height:80px }
.small-title{ font-weight:600; color:#111827 }
.small-body{ display:flex; align-items:center; gap:12px; justify-content:space-between }
.circle-ind{ width:48px; height:48px; border-radius:50%; background:linear-gradient(180deg,#dffaff,#bfe9ff); display:flex; align-items:center; justify-content:center; color:#065f8d; font-weight:700 }
.filter-bar{ height:10px; width:120px; background:#f1f5f9; border-radius:6px; overflow:hidden }
.filter-fill{ height:100%; background: linear-gradient(90deg,#ffd05b,#ff8a5b); transition: width 800ms ease }

/* right column */
.col.right-col{ width:320px; display:flex; flex-direction:column; gap:14px }
.metric-card{ padding:14px; border-radius:12px; background:var(--card); box-shadow:var(--shadow) }
.metric-title{ color:var(--muted); font-size:13px; margin-bottom:8px }
.metric-value{ font-size:20px; font-weight:700; color:#111827 }
.metric-sub{ color:var(--muted); font-size:12px; margin-top:6px }
.metric-bar{ height:8px; background:#eef2ff; border-radius:8px; overflow:hidden; margin-top:10px }
.metric-fill{ height:100%; background: linear-gradient(90deg,#26a9ff,#2f69f6); transition: width 700ms ease }

/* controls column */
.controls-col{ display:flex; gap:8px; margin-top:8px }
.btn.small{ padding:6px 8px; border-radius:8px }
.btn.small.outline{ border:1px solid #e6e9ee; background:transparent }

/* responsive */
@media (max-width: 1100px){
  .rdash-main{ flex-direction:column }
  .col.mid-col, .col.right-col{ width:100% }
  .big-card{ min-width: auto }
}
Installation & usage
Create React app (if you don't already have one)

perl
Copy
Edit
npx create-react-app my-app
cd my-app
Save files:

src/Dashboard.jsx ← component code above

src/Dashboard.css ← stylesheet above

In src/App.js import & use:

jsx
Copy
Edit
import React from "react";
import Dashboard from "./Dashboard";

function App(){
  return <Dashboard />;
}

export default App;
Run:

sql
Copy
Edit
npm install
npm start
