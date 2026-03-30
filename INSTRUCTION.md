# SF-CORE.ANCHOR v1.0 — SYSTEM EXPANSION SPEC (OUT-OF-BROWSER EXECUTION)

## 1. SYSTEM DEFINITION

SF-CORE.ANCHOR is no longer a browser-contained simulation.
It must be refactored into a **real-world state orchestrator** with three active domains:

* Observable Inputs (real sensors)
* Deterministic Evaluation (NTI + Rules Engine)
* Real Actuation (filesystem, processes, external systems)

The browser becomes **read-only visualization**, not execution layer.

---

## 2. TARGET ARCHITECTURE

### 2.1 High-Level Layers

```
[ SENSORS ] ──► [ CORE ENGINE ] ──► [ ACTUATORS ]
                      │
                      ▼
               [ STATE STORE ]
                      │
                      ▼
               [ UI (Dashboard) ]
```

---

## 3. CORE ENGINE (MANDATORY)

### 3.1 State Object (single source of truth)

```python
State = {
    "commits": int,
    "focus_minutes": int,
    "sensorIn": float,
    "mihm": float,
    "nti": float,
    "active_tool": str | None,
    "rules_active": list,
    "timestamp": datetime
}
```

---

### 3.2 NTI Calculation (must remain deterministic)

```
sensorIn = commits_norm * 0.55 + focus_norm * 0.45
mihm     = time_based_projection()
nti      = abs(mihm - sensorIn)
```

No randomness allowed.

---

### 3.3 Rule Engine (event-driven)

Rules trigger ONLY on **rising edge condition**.

```python
if rule.condition(state) and not rule.was_active:
    rule.execute(state)
```

Persist rule activation state to prevent loops.

---

## 4. SENSORS (REPLACE ALL MANUAL INPUT)

### 4.1 Git Activity Sensor

Source:

* Git CLI OR GitHub API

Output:

```json
{ "commits_today": int }
```

---

### 4.2 Focus Time Sensor

Priority order:

1. OS-level tracking (preferred)
2. External API (RescueTime / Toggl)
3. Fallback: manual input file

Output:

```json
{ "focus_minutes": int }
```

---

### 4.3 Time Projection (MIHM)

```python
def mihm(now):
    h = now.hour + now.minute/60
    if h <= 6: return 0
    if h >= 22: return 1
    return (h - 6) / 16
```

---

## 5. ACTUATORS (REAL EXECUTION REQUIRED)

### 5.1 File System Actions

* Write logs
* Create snapshots
* Persist state

Paths:

```
/logs/YYYY-MM-DD.log
/snapshots/
/state/state.json
```

---

### 5.2 Git Actions

Trigger real commits:

```bash
git add .
git commit -m "auto: snapshot NTI=0.83"
```

---

### 5.3 System Commands

Must support:

* Run shell scripts
* Open / close applications
* Trigger notifications

Example:

```bash
osascript -e 'quit app "Twitter"'
```

(or platform equivalent)

---

### 5.4 Notification Layer

* Console output (mandatory)
* Desktop notification (optional)
* Sound trigger (optional)

---

## 6. EXECUTION ENGINE

### 6.1 execAction MUST BE REPLACED

Current (invalid):

* UI log only

Required:

```python
def exec_action(type, description, result):
    write_log(type, description, result)
    persist_state()
    if type == "COMMIT":
        run_git_commit(description)
    if type == "EJECUTAR":
        run_script(description)
```

---

## 7. STATE PERSISTENCE

### 7.1 Required

* Save state every cycle (1s–5s)
* Recover state on restart

Format:

```json
{
  "nti": 0.42,
  "mihm": 0.55,
  "sensorIn": 0.13,
  "last_update": "ISO8601"
}
```

---

## 8. INTERFACE SPLIT

### 8.1 Backend (mandatory)

Options:

* Python (FastAPI / Flask)
* Node.js (Express)

Responsibilities:

* Compute state
* Execute rules
* Trigger actuators
* Expose API

---

### 8.2 Frontend (existing HTML)

Refactor:

* Remove logic from JS
* Replace with API polling:

```javascript
fetch('/state')
```

Frontend becomes:

* NTI visualization
* Rule status
* Logs viewer

NO decision logic in browser.

---

## 9. API CONTRACT

### GET /state

```json
{
  "nti": float,
  "mihm": float,
  "sensorIn": float,
  "active_tool": string,
  "rules_active": []
}
```

---

### GET /logs

Returns last N execution events.

---

### POST /command

```json
{ "command": "commit test" }
```

---

## 10. TOOL SYSTEM (KEEP STRUCTURE, CHANGE EFFECT)

Tools must trigger real actions:

| Tool    | Effect                |
| ------- | --------------------- |
| gavel   | force decision script |
| chisel  | write narrative log   |
| compass | recalibrate target    |
| level   | normalize NTI         |

---

## 11. LOOP EXECUTION

Main loop (1s interval):

```python
while True:
    state = read_sensors()
    state.mihm = compute_mihm()
    state.nti  = abs(state.mihm - state.sensorIn)
    evaluate_rules(state)
    persist_state(state)
    sleep(1)
```

---

## 12. PRIORITY ORDER (NON-NEGOTIABLE)

1. Replace execAction with real execution
2. Replace sliders with real sensors
3. Persist state
4. Move logic to backend
5. Convert UI into dashboard

---

## 13. FAILURE CONDITIONS

System is considered non-functional if:

* NTI does not affect real world
* Rules do not trigger external actions
* State is not persisted
* Inputs are manual

---

## 14. SUCCESS CONDITION

System is valid when:

* NTI changes → triggers real consequences
* Logs exist outside browser
* Commits are generated automatically
* Sensors reflect actual behavior

---

## 15. IMPLEMENTATION MODE

Claude / Codex must:

* Refactor codebase into backend + frontend
* Create executable modules (not mock)
* Avoid placeholders unless explicitly required
* Prioritize working system over theoretical purity

No UI improvements until system has real-world effect.

---

END OF SPEC
