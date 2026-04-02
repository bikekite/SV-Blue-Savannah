
**Timing constants**
Two constants at the top define the cycle — `ON_DELAY_MINUTES` controls how long the output stays ON, and `OFF_DELAY_HOURS` controls how long it stays OFF. These are converted to milliseconds internally so JavaScript's `setTimeout` can use them.

---

**The cycle**
Once started, the timer runs a continuous OFF → ON → OFF loop using two mutually scheduling functions — `doOff()` and `doOn()`. Each one sends its signal, then schedules the other to run after its respective delay. This creates the repeating cycle without any external clock or tick.

---

**State tracking**
Two pieces of state are stored in Node-RED's `context` (persistent memory inside the function node):
- `enabled` — a boolean that tells the timer whether it should keep running. Both `doOn()` and `doOff()` check this at the top before doing anything, so the cycle can be stopped cleanly even if a timeout is already waiting.
- `onTimer` / `offTimer` — references to the active timeouts so they can be cancelled at any time via `clearTimers()`.

---

**Inputs**
All four inject nodes feed into the single input and are routed by `msg.topic`:
- `reset` — clears any running timers and restarts the cycle fresh from the OFF phase
- `enable` — `true` starts the cycle, `false` clears timers, sets `enabled` to false, and sends a final OFF signal so the output doesn't get stuck ON
- `bypass` — behaviour splits based on current state. If the timer is **running**, it fires ON instantly for `ON_DELAY` duration then sends OFF and restarts the cycle. If the timer is **stopped**, it fires ON once for `ON_DELAY` duration then sends OFF and remains stopped — `enabled` is never set to true so the cycle doesn't begin looping.

---

**Outputs**
- Output 1 carries the ON signal (`payload: 1`)
- Output 2 carries the OFF signal (`payload: 0`)
- Output 3 carries a human-readable status string for monitoring in the debug panel
