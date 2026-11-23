---
name: aha
description: >-
  Transform a single technical Markdown knowledge doc into a deeply unpacked,
  genuinely animated, interactive offline HTML micro-course. Engineered for
  Devin + fast coding models via a mandatory 5-phase file-backed pipeline
  (Expand -> Storyboard -> Hybrid Modular Build -> Offline Bundle -> Playwright
  Self-Audit) that prevents summarization laziness, fake controls, and SVG bugs.
---

# Aha v2.0 — Self-Bootstrapping Interactive Engineering Micro-Course Pipeline

Mission: Turn dense engineering notes into an **active interactive lab** where a non-expert or junior engineer truly internalizes the underlying mechanism—not by memorizing script names, but by **watching causality move, breaking the system, and testing predictions**.

> **CRITICAL EXECUTION RULE (NO ONE-SHOT HTML)**
> You are STRICTLY FORBIDDEN from generating the final `.html` in a single pass.
> You MUST execute all 5 phases sequentially using real files inside `./aha_work/`, and you MUST NOT deliver the final HTML until Phase 5 (`audit_aha.py`) exits with code `0` and you have visually inspected the generated screenshots.

---

## Phase 1: Deep Dissection & Mandatory Knowledge Expansion
**Input**: The source knowledge `.md` file.
**Output**: `./aha_work/01_expansion.md`

Never summarize or copy-paste the source `.md` into bullet lists. Your job is to **UNPACK and EXPAND** the source facts into teachable engineering intuition.
- **Fact Integrity Rule**: Never invent project-specific filenames, line numbers, key names, or byte offsets not present in the source `.md`.
- **Pedagogical Expansion Rule**: You MUST actively enrich the explanation using first-principles CS/security/hardware knowledge across these **6 Mandatory Expansion Blocks**:
  1. **The Naive Design Counterfactual ("Why not the dumb way?")**: Show the obvious, simple design an engineer would try first—and demonstrate concretely how it fails (latency explosion, single point of compromise, key reuse catastrophe, deadlock, etc.).
  2. **Grounded Mental Model + Analogy Boundary**: Provide 1 crystal-clear real-world analogy AND explicitly state 1 place where the analogy stops matching hardware/software reality.
  3. **Micro-Causal Step Breakdown (Forward & Reverse)**: Unpack every black-box sentence into 4–6 sequential micro-ticks (`Tick 0 -> Tick N`: Input -> Operation -> State Mutation -> Output). If the doc only explains build/encode time, also explain the **runtime/decode/verify reverse path** conceptually (clearly labeled as conceptual runtime flow if not detailed in source).
  4. **Break-It / Attack / Failure Scenarios (Min 2 Scenarios)**: Define at least 2 concrete failure injections the reader can trigger in the lab (e.g., *Tamper 1 bit in payload*, *Mismatch DEV/PROD key*, *Compromise Build Server plaintext*, *Power-loss mid-update*) and trace the exact step where the system blocks or fails.
  5. **Common Misconceptions (Min 3 Traps)**: Write 3 "90% of engineers assume X, but actually Y" callouts.
  6. **Dual-Layer Separation & Glossary (Min 6 Terms)**:
     - **Intuition Track (Primary Prose)**: Mentor-at-whiteboard tone (`"你想想看，如果直接..."`). Explain *why* and *how it behaves* before mentioning file names.
     - **Source Trace Track (Collapsible Drawer)**: Move raw file paths, line numbers, CLI flags, and hex literals here so they remain 100% accessible without choking the main narrative.
     - **Glossary**: Define at least **6** distinct technical terms that appear as clickable `.term-chip` elements across the stages.

---

## Phase 2: Interactive Choreography Blueprint
**Input**: `./aha_work/01_expansion.md`
**Output**: `./aha_work/02_blueprint.json`

Design 5–6 progressive stages following the escalation arc:
- **Stage 0**: Pain Point & Naive vs. Real Architecture (interactive comparison)
- **Stage 1**: Core Anatomy / Components (interactive inspection + live data/key relationship)
- **Stage 2**: Step-by-Step Mechanism Flow (animated playback: `Play / Pause / Step / Reset`)
- **Stage 3**: Attack, Tamper, or Edge-Case Lab (**Predict-Before-Run quiz** + live fault injection)
- **Stage 4**: Production / Migration / Trade-off Diff (interactive mode switcher that **visibly transforms** architecture boundaries, risk badges, and animated paths)

### Anti-Static-Slide & Anti-Fake-Control Rules
1. **No Dead Controls**: Every mode pill, toggle, slider, or fault button MUST explicitly list in `02_blueprint.json` which DOM nodes, badges, wires, metrics, and animations change when toggled. If a stage does not need a global switcher, give it a **stage-specific control** (e.g., *Inject 1-Bit Tamper*, *Simulate Build Server Leak*, *Step Through Boot Verify*).
2. **Real Motion Required on Every Stage**: Static boxes with hover text are NOT enough. Every stage MUST implement at least one genuine visual dynamic:
   - **Animated Token/Packet Flow**: Colored data/key tokens traveling along SVG paths (`Engine.animatePacket(pathId, label, color, duration)`).
   - **Step-by-Step Playback**: A `Play / Pause / Prev / Next / Reset` bar highlighting active nodes, pulsing active wires, and updating a live step caption + metric bar.
   - **Live Tamper / State Propagation**: Clicking a tamper/fault button flips a bit or leaks a key, triggering a red domino pulse along the path and lighting up the exact rejection/breach node.
   - **Boundary Transformation**: Toggling architecture modes (e.g., `DEV` vs `PROD Current` vs `PROD HSM`) visibly moves risk indicators, switches wire routing (Local vs mTLS HSM), updates latency/security meters, and changes active code snippets.
3. **Predict-Before-Run (Min 2 Stages)**: Include an inline 1-click prediction card (*"動手前先猜猜看：如果..."* with 2–3 choices). Clicking an option reveals immediate feedback AND auto-runs the corresponding animation to prove it visually.

---

## Phase 3: Hybrid Modular Build (HTML Cards + SVG Dynamic Overlay)
**Output Files**:
- `./aha_work/shell.html` (CSS light engineering theme, Dual-Pane layout, Stepper, Playback bar, Predict card, Collapsible Source Trace, Term Drawer, Bottom HUD)
- `./aha_work/engine.js` (Hybrid layout wire router & packet animation micro-engine)
- `./aha_work/stages/stage_0.js` ... `./aha_work/stages/stage_N.js` (**ONE separate file per stage**, ~150–280 lines each to guarantee peak code precision)

### Why Hybrid Layout Is Mandatory (Zero Text Overflow, Zero Broken SVG)
- **NEVER hand-draw text cards inside pure SVG `<rect>` + `<text>`.**
- Render all zones, containers, badges, bit-cells, and node cards as **semantic HTML/CSS Flexbox/Grid elements** inside `.stage-board` (position: relative). HTML wraps text automatically, expands card height naturally, and guarantees reliable hover/click events.
- Place a single transparent `<svg class="wire-layer">` (`position: absolute; inset: 0; pointer-events: none;`) on top/behind the HTML cards ONLY for **dynamic curved connectors, arrowheads, and animated data/key particles**.
- Use the following micro-engine pattern in `engine.js` so wires automatically anchor to real HTML card positions via `getBoundingClientRect()` (and re-route on `window.resize`):

```javascript
// Mandatory core helper in engine.js (adapt & extend cleanly)
const AhaEngine = {
  drawWires(boardEl, svgEl, wires) {
    // Preserve <defs> markers! NEVER destroy <defs> with blind innerHTML.
    const bRect = boardEl.getBoundingClientRect();
    svgEl.setAttribute('viewBox', `0 0 ${bRect.width} ${bRect.height}`);
    svgEl.querySelectorAll('.dyn-layer').forEach(n => n.remove());
    const g = document.createElementNS('[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)', 'g');
    g.setAttribute('class', 'dyn-layer');
    wires.forEach(w => {
      const a = boardEl.querySelector(`[data-node="${w.from}"]`);
      const b = boardEl.querySelector(`[data-node="${w.to}"]`);
      if (!a || !b) return;
      const ar = a.getBoundingClientRect(), br = b.getBoundingClientRect();
      // Compute anchor points relative to boardEl (support right->left, bottom->top, etc.)
      const x1 = (w.fromSide === 'bottom' ? ar.left + ar.width/2 : ar.right) - bRect.left;
      const y1 = (w.fromSide === 'bottom' ? ar.bottom : ar.top + ar.height/2) - bRect.top;
      const x2 = (w.toSide === 'top' ? br.left + br.width/2 : br.left) - bRect.left;
      const y2 = (w.toSide === 'top' ? br.top : br.top + br.height/2) - bRect.top;
      const dx = Math.max(40, Math.abs(x2 - x1) * 0.45);
      const d = (w.fromSide === 'bottom')
        ? `M ${x1} ${y1} C ${x1} ${y1+36}, ${x2} ${y2-36}, ${x2} ${y2}`
        : `M ${x1} ${y1} C ${x1+dx} ${y1}, ${x2-dx} ${y2}, ${x2} ${y2}`;
      const p = document.createElementNS('[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)', 'path');
      p.setAttribute('id', `wire-${w.id}`);
      p.setAttribute('d', d);
      p.setAttribute('class', `wire ${w.style || ''} ${w.active ? 'wire-active' : ''}`);
      p.setAttribute('marker-end', `url(#arrow-${w.color || 'slate'})`);
      g.appendChild(p);
    });
    svgEl.appendChild(g);
  },
  animatePacket(svgEl, wireId, label, color = '#0284C7', duration = 900) {
    const path = svgEl.querySelector(`#wire-${wireId}`);
    if (!path) return;
    const len = path.getTotalLength();
    const g = document.createElementNS('[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)', 'g');
    g.setAttribute('class', 'dyn-layer packet-token');
    g.innerHTML = `<circle r="6" fill="${color}" stroke="#fff" stroke-width="2"></circle>`
      + (label ? `<rect x="10" y="-11" width="${Math.max(44, label.length*7+14)}" height="20" rx="4" fill="${color}"></rect>`
               + `<text x="17" y="3" fill="#fff" font-size="11" font-weight="700" font-family="ui-monospace,monospace">${label}</text>` : '');
    svgEl.appendChild(g);
    const start = performance.now();
    const tick = (now) => {
      const t = Math.min(1, (now - start) / duration);
      const ease = t < 0.5 ? 2*t*t : -1 + (4 - 2*t)*t;
      const pt = path.getPointAtLength(len * ease);
      g.setAttribute('transform', `translate(${pt.x}, ${pt.y})`);
      if (t < 1) requestAnimationFrame(tick);
      else setTimeout(() => g.remove(), 350);
    };
    requestAnimationFrame(tick);
  }
};
