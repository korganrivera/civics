# Simulated Runtime Framework — compressed, complete

Fast feedback loop; no hardware dependencies; enables isolated stress-testing of `auren-capture`, `auren-parse`, `auren-reflect`; prepares structure for later integration into lab rig or field hardware.

---

## Deliverables to generate next

1. **Simulated Member + Tool Stream (`mock_stream.py`)**

   * Emits pseudo-random events with these fields: `timestamp` (UTC ISO), `member` (random from set), `event_type` (one of `tool_use`, `mood_shift`, `proximity_change`, `silence`), `tool` (random from tools set), `mood` (one of moods), `proximity` (random member).
   * Purpose: simulate voice mood markers, NFC-like tool use, BLE-like proximity.

2. **Initial Parser + Reflector Hooks (`auren-parse.py`, `auren-reflect.py`)**

   * `auren-parse.py`:

     * `parse_event(event)` → builds log entry with `id`, `timestamp`, `member`, `trigger`, `tool`, `emotion`, `proximity`, `inferred_event`, `suggested_action`.
     * `infer_event(event)` rules:

       * `mood_shift` + mood in `["tense","foggy"]` → `"potential rupture"`.
       * `tool_use` → `"role execution"`.
       * `silence` → `"missed reflection"`.
       * otherwise → `"neutral"`.
     * `suggest(event)` rules:

       * `mood_shift` with `tense|foggy` → `"invite micro-repair"`.
       * `silence` → `"prompt for trace"`.
       * otherwise → `None`.
     * `write_log(entry, path="auren-log.yaml")` appends entry to YAML array (creates file if missing).
     * Example test: import `generate_event` from `mock_stream` and parse/write several events.

   * `auren-reflect.py`:

     * `reflect_on_log(path="auren-log.yaml")` loads YAML, takes last 5 entries, selects entries where `suggested_action` exists, returns list of objects `{member, action, reason, timestamp}`.
     * CLI/test prints each suggestion as: `[timestamp] member → action (reason)`.

3. **Run Loop Script (`auren-sim.py`)**

   * Imports `generate_event`, `parse_event`, `write_log`, `reflect_on_log`.
   * `run_sim_loop(cycles=5, delay=2)`:

     * for each cycle: generate event, print event, parse and write log, sleep `delay` seconds.
     * After cycles, call `reflect_on_log()` and print suggestions.
   * Intended runtime: 10-minute simulated ticks in original spec; scaffold uses configurable cycles/delay.

---

## Request (user instruction)

Proceed to generate Simulated Runtime Framework starting with:

* `mock_stream.py` (fake data emitter)
* `auren-parse.py` (event → YAML translator)
* `auren-reflect.py` (LLM-based suggestion engine)

Start system loop when ready. Confirm to begin coding scaffold modularly so real hardware can plug in later.

---

## Activation confirmation and scaffold (condensed)

Simulator activation confirmed. Files scaffolded and behavior implemented:

**`mock_stream.py`** — key elements:

* `members = ["kai","rhea","sol","jun"]`
* `tools = ["shovel_1","pot_3","sensor_pad","light_rig"]`
* `moods = ["calm","tense","weary","focused","foggy"]`
* `events = ["tool_use","mood_shift","proximity_change","silence"]`
* `generate_event()` returns dict with `timestamp` (UTC ISO), `member`, `event_type`, `tool`, `mood`, `proximity`.
* If run as `__main__`, emits 10 events printing each with 1s sleep.

**`auren-parse.py`** — implements `parse_event`, `infer_event`, `suggest`, `write_log` as described above; test block imports `generate_event` and writes 5 parsed logs to `auren-log.yaml`.

**`auren-reflect.py`** — implements `reflect_on_log` that reads `auren-log.yaml`, selects recent entries with `suggested_action`, and returns/prints member-action-reason-timestamp objects.

**`auren-sim.py`** — ties modules into a run loop:

* Calls `generate_event()`, `parse_event()`, `write_log()` each cycle; after cycles calls `reflect_on_log()` and prints suggestions.
* Example `run_sim_loop(cycles=5, delay=2)` provided.

---

## Next-phase options (available upgrades)

1. Ritual Timing + Role Drift Detection (`auren-scheduler.py`) — flags missed rituals, tracks tool-use vs expected roles, logs `role_drift` when drift exceeds threshold, suggests sync repair; adds `auren-scheduler.py`, expanded schema for expected rituals/role norms.
2. Markdown/Console Daily Export (`auren-export.py`) — auto-generate readable daily summaries: rituals completed/missed, mood trends, rupture/repair events, suggested prompts.
3. Sensor Input Bridge (`sensor_bridge.py`) — swap `generate_event()` for real inputs: BLE/Bluetooth proximity, mic sentiment (Whisper+EmotionNet), NFC/RFID tool use, manual YAML via QR/mobile.
4. Ritual Companion Integration — link `auren-lite-ritual-agent.md` to runtime for guided sessions, real-time reflection, micro-repair triggers.
5. Custom Fork — user-defined spec.

Decision prompt: choose A–E or give custom spec.

---

## Selected path and scheduler deployment (recorded choices and outputs)

User selected: **A. Ritual Timing + Role Drift**. Module generated and deployed with these specs:

**auren-scheduler.py — core capabilities**

* Ritual Clock:

  * Tracks expected daily/weekly rituals, flags missed/delayed events, emits `ritual_missed:<ritual_name>`.
* Role Drift Detector:

  * Compares actual tool use/presence to known roles; if drift > threshold (default 2 cycles) logs `role_drift:<member_id>:<role>` and suggests sync repair via `auren-reflect`.
* Sync Repair Queue Generator:

  * Emits active ruptures list (drift, silence, mood shift) and pre-generates LLM prompt for `auren-reflect`.

**Integration hooks**

* Consumes: `auren-log.yaml`, `mock_stream.py` (or sensor feed), `auren-config.yaml` (expected rituals, roles).
* Emits: `scheduler-status.json` (active drift, missed rituals), `scheduler-events.yaml` (for `auren-reflect.py`), optional `ritual-prompt.md` for handoff to ritual agent.

**Sample `auren-config.yaml`**

```yaml
ritual_schedule:
  daily_trace: "08:00"
  sync_checkin: "Saturday 14:00"

expected_roles:
  - member: "Kai"
    role: "Steward"
    tools: ["soil_monitor", "water_log"]
  - member: "Rae"
    role: "Cook"
    tools: ["kitchen_scale", "heat_sensor"]
```

**Deployment parameters confirmed by user**

1. Runtime tick granularity: **1 hour** (default)
2. Drift threshold: **2** consecutive missing role-tool matches (balanced)
3. Initial `auren-config.yaml`: **sample** values used (Kai = Steward, Rae = Cook)

Result: `auren-scheduler.py` generated and deployed; it runs hourly-tick simulation and integrates with existing loop.

---

## Final note (factual)

* The simulated runtime scaffold is live and modular; real sensor feeds can replace `mock_stream.generate_event()`; options listed for next-phase features remain available.

