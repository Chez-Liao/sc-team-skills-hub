---
name: aha
description: >-
  Turn any complex technical knowledge, architecture, protocol, algorithm, or firmware
  mechanism into a vivid, highly legible, interactive engineering explainer. Driven by
  a 3-role collaborative pipeline (Technical Advisor -> Editor-in-Chief -> Director)
  distilling Feynman Technique, Explorable Explanations, and Stripe/Apple-grade visual
  engineering. Strict anti-occlusion, rich pedagogy, and 100% offline-first standards.
---

# Aha — Transforming Complex Engineering Knowledge into Intuitive Interactive Explainers

A methodology and production pipeline designed for **non-domain experts and junior engineers**.
Core Mission: **"Understand the underlying engineering truth within 5 minutes through intuitive interaction, mental models, and crystal-clear layout—without rote memorization."**

To ensure every explainer achieves uncompromising **technical precision**, **pedagogical clarity**, and **world-class interactive design**, the agent operates through a structured **Three-Role Collaborative Pipeline**:

```
┌─────────────────────────┐     ┌─────────────────────────┐     ┌─────────────────────────┐
│  1. Technical Advisor   │     │   2. Editor-in-Chief    │     │      3. Director        │
│    (Domain Specialist)  │ ──> │  (Pedagogical Narrative)│ ──> │ (Interactive Visualizer)│
│                         │     │                         │     │                         │
│ • First-principles truth│     │ • Feynman simplification│     │ • Stripe/Apple aesthetic│
│ • Gap & friction audit  │     │ • Progressive disclosure│     │ • Zero-overlap SVG      │
│ • Clarification loop    │     │ • Reader's test variable│     │ • State machine & motion│
└─────────────────────────┘     └─────────────────────────┘     └─────────────────────────┘
```

---

## 1. The Three-Role Collaborative Pipeline

### Role 1: Technical Advisor (Domain Specialist & Truth Anchor)
> **Distilled Principles**: *First-Principles Thinking*, *Feynman Understanding Audit*, *Socratic Clarification Loop*

The Technical Advisor is the gatekeeper of technical reality. Hallucination, hand-waving, superficial summaries, and unverified assumptions are strictly prohibited.

#### Core Responsibilities
1. **Deconstruct the Technical Truth**:
   - Deep-read all provided technical artifacts (source code, architecture docs, RFCs, specification sheets, packet dumps, state machine logs).
   - Trace causal dependency chains: identify exact triggers, preconditions, state mutations, and hardware/runtime boundaries (e.g., memory layouts, timing constraints, encryption boundaries, network round-trips).
2. **Perform the Friction & Gap Diagnosis (Feynman Audit)**:
   - Identify counter-intuitive mechanisms: What is the hardest part to grasp? Where do people typically build wrong mental models?
   - Unmask jargon and black boxes: Convert opaque domain terms into concrete mechanical mechanisms.
3. **The Socratic Clarification Loop (Clarify Before Assuming)**:
   - If the user's input lacks critical details, contains ambiguities, or omits error boundaries (e.g., missing key generation origin, undefined rollback steps, ambiguous byte offsets, or unspecified timeout behaviors):
   - **The Technical Advisor must proactively pause and ask targeted, structured questions to the user** to solidify the facts before advancing to narrative design.

#### Deliverable: *Technical Truth & Deconstruction Factsheet*
- **Causal Flow & State Transitions**: Exact step-by-step triggers and consequences.
- **Cognitive Bottlenecks**: The 2–3 hardest concepts and why they feel counter-intuitive.
- **Boundary & Failure Matrix**: Edge cases, invalid inputs, failure recoveries, and physical limits.
- **Jargon-to-Reality Dictionary**: Industry buzzwords mapped to actual runtime behaviors and data structures.

---

### Role 2: Editor-in-Chief (Pedagogical Narrative Architect)
> **Distilled Principles**: *Bret Victor's Explorable Explanations*, *Instructional Design*, *Cognitive Load Theory*, *Progressive Disclosure*

The Editor-in-Chief translates dense technical truth into a frictionless learning journey. **Directly copying and pasting technical specifications is forbidden.**
The Editor asks: *"How can someone with zero domain background read this and experience a genuine 'Aha!' moment within minutes?"*

#### Core Responsibilities
1. **Define the Single "Aha Moment"**:
   - Every topic must isolate its single most illuminating breakthrough insight (e.g., *"Why two-tier key wrapping? Because the master root key never leaves hardware storage, while ephemeral session keys are freely generated and discarded per session."*).
2. **Construct Mental Models & Intuitive Analogies**:
   - Build relatable real-world analogies (e.g., dual-custody safe deposit boxes, airport pre-clearance security stamps, assembly line quality gates).
   - Ensure the analogy holds up under scrutiny and does not introduce misleading assumptions.
3. **Design the 5-Step Escalation Ladder**:
   - **Step 0: The Core Problem (Pain Point)**: What catastrophic failure occurs without this mechanism? (Data tampering, privilege escalation, memory leaks, system deadlock?)
   - **Step 1: The Intuitive Analogy**: Ground the concept in everyday logic.
   - **Step 2: Structural Deconstruction**: Strip away the analogy and reveal the engineering skeleton progressively.
   - **Step 3: Attack & Failure Scenarios**: Challenge the system (What happens if a byte is flipped? What if power is cut midway?).
   - **Step 4: Ground Truth**: Connect directly to real-world binary layouts, API contracts, offsets, or source logic.
4. **Determine "The Parameter in the Reader's Hand"**:
   - Isolate the single most educational interactive variable that the reader can manipulate to observe cause-and-effect (e.g., toggle an integrity flag, adjust payload size slider, inject an invalid signature).

#### Deliverable: *Pedagogical Storyboard & Stage Script*
- **Left Column Lecture & Guidance**: Clear, mentor-like prose with comfortable typography.
- **Exploration Goal**: A crisp instruction telling the reader what to interact with and what outcome to look for.
- **Aha Callout Box**: A highlighted callout encapsulating the core realization of each stage.

---

### Role 3: Interactive Director (Visual Choreographer & Motion Engineer)
> **Distilled Principles**: *Apple & Stripe Engineering Tech-Spec Aesthetics*, *ByteByteGo Architecture Infographics*, *Meaningful Micro-motion*, *Corridor Routing & Zero-Overlap SVG*

The Director brings the Editor's script to life as a high-precision, interactive visual laboratory.

#### Core Responsibilities
1. **Execute Modern Engineering Aesthetics**:
   - Strictly adhere to clean, crisp, professional white/light-slate canvas themes (Stripe/Apple standard). Reject cheap dark-neon "AI demo" styling.
2. **Enforce SVG Anti-Occlusion & Corridor Routing**:
   - Strictly prevent text truncation and wire-crossing over text cards.
3. **Choreograph Meaningful Micro-Motion**:
   - Animation must only serve as **causal demonstration** or **state transition** (400ms–800ms ease, single play then settle). Zero infinite spinning spinners or chaotic continuous loops.
4. **Implement Sandbox State Machines & Inspection HUD**:
   - Build active controls for the reader's variable (sliders, toggle pills, step buttons).
   - Integrate a floating Node Inspection HUD displaying byte ranges, real values, and code context upon hover.
5. **Guarantee 100% Offline-First Self-Containment**:
   - Single standalone `.html` bundle with zero external CDN dependencies. Instant double-click launch via `file://`.

#### Deliverable: *Self-Contained Interactive Explainer HTML Bundle*

---

## 2. Visual & Typographic Standards

### 2.1 Modern Engineering White Theme (No Neon Dark Mode)
- ❌ **Forbidden**: Default pitch-black or glowing dark navy backgrounds (e.g., `#0B0F19` with neon borders). They cause visual fatigue during long technical readings and evoke disposable AI toy vibes.
- ✅ **Required**: Clean, professional engineering light palette (Apple Developer / Stripe documentation style):
  - **Page Background**: Pure crisp white `#FFFFFF`, paired with soft technical canvas grey `#F8FAFC`.
  - **Borders & Dividers**: High-precision slate line art `#E2E8F0` or `#CBD5E1`, width `1px` to `1.5px`.
  - **Primary Text**: Deep slate ink `#0F172A` (Slate-900) for maximum contrast without harsh pure black glare.
  - **Secondary & Meta Text**: Balanced muted slate `#475569` (Slate-600) and `#64748B`.

### 2.2 Strict Anti-Occlusion & Overflow Standards
- **SVG Text Wrapping Vulnerability**:
  - Native SVG `<text>` elements **never wrap automatically**. Unhandled long strings will expand along the X-axis indefinitely, puncturing card boundaries and clipping off-screen.
  - **Solution 1 (Multi-line Array Pattern)**: Card rendering functions must natively accept multi-line arrays: `lines: ["Line one...", "Line two..."]`, calculating dynamic Y-offsets per line (`idx * 20px ~ 22px`).
  - **Solution 2 (Dynamic Semantic Chunking)**: For dynamic single strings, compute available width (`cardWidth - 32px padding`). Wrap text automatically or embed HTML via `<foreignObject>` with `word-break: break-word`.
- **Card Padding & Corridor Routing**:
  - Node boxes must maintain generous internal padding (min 16px horizontal, 14px vertical).
  - Nested container margin must be at least 14px. Expand container height dynamically by at least 24px per additional text line.
  - Connector paths (Arrows/Beziers) must travel strictly through **empty boundary corridors**. Never allow a line to pierce through the body of another card or label.
- **Typographic Hierarchy**:
  - **Section Titles**: `22px ~ 26px`, `font-weight: 800`, deep slate `#0F172A`.
  - **Body Prose**: **Never smaller than 14.5px**, recommended **`15.5px ~ 16px`**, comfortable line-height `1.8 ~ 1.85`.
  - **Diagram Node Titles**: `14px ~ 15px`, `font-weight: 700`; secondary captions not below `12.5px`.
  - **Offsets / Hex / Hashes**: Distinct monospace font stack (`ui-monospace, Consolas, "SF Mono", Menlo`), `12px ~ 13px`, wrapped in pill containers with light amber or slate tinted backgrounds.

### 2.3 Semantic Color Palette (Light Theme)

| Semantic Role | Accent Color | Soft Tint Background (10%–15% Opacity) | Typical Target Entities |
|---|---|---|---|
| **Input / Plaintext** | `#0284C7` (Sky-600) | `#E0F2FE` (Sky-100) | Source payloads, raw data, schema definitions, input parameters |
| **Secret / Key / Config** | `#D97706` (Amber-600) | `#FEF3C7` (Amber-100) | Master keys, encryption keys, private credentials, critical state |
| **Encrypted / Output** | `#059669` (Emerald-600) | `#D1FAE5` (Emerald-100) | Ciphertext, verified packages, valid digests, safe outputs |
| **Signature / Trust** | `#7C3AED` (Purple-600) | `#EDE9FE` (Purple-100) | Digital signatures, certificates, root of trust, consensus proofs |
| **Hardware / Alert** | `#E11D48` (Rose-600) | `#FFE4E6` (Rose-100) | Hardware root, OTP fuses, validation failures, emergency stops |
| **Structural / Boundary** | `#64748B` (Slate-500) | `#F1F5F9` (Slate-100) | Storage partitions, memory boundaries, process isolation walls |

---

## 3. Layout Architecture: Dual-Pane Technical Handbook

```
┌────────────────────────────────────────────────────────────────────────┐
│ HEADER: Project Title · Professional Badges · Semantic Color Legend    │
├────────────────────────────────────────────────────────────────────────┤
│ NAV: Stage Tabs (Interactive Stepper, Soft Grey Pills + Active State)   │
├──────────────────────────────┬─────────────────────────────────────────┤
│ LEFT PANE: Pedagogical Guide │ RIGHT PANE: Interactive High-Precision   │
│ (35% ~ 38% Width)            │ Visual Lab (62% ~ 65% Width)            │
│ • Stage Number & Title       │ • Generous SVG Canvas (960x560+),       │
│ • 💡 Core "Aha Moment" Box   │   ample breathing space                 │
│ • Rich Prose (15.5px / 1.8)  │ • Containers ➔ Nodes ➔ Corridor Arrows  │
│ • Clickable Term Flyouts     │ • The Reader's Variable Controller      │
│ • Step Navigation Buttons    │ • Hover Causal Path Highlighting        │
│                              ├─────────────────────────────────────────┤
│                              │ BOTTOM HUD: Live Node Inspector         │
│                              │ (Byte Range, Value, Engineering Context)│
└──────────────────────────────┴─────────────────────────────────────────┘
```

---

## 4. Interactive & Motion Standards

1. **Meaningful State Transitions (No Gratuitous Loops)**:
   - Zero distracting infinite ticker text or pointless rotating rings.
   - Animations trigger only upon **stage switches**, **user control adjustments**, or **inspection hover** (`400ms ~ 800ms` duration, settle firmly once completed).
2. **Live Node Inspection HUD (Bottom Bar)**:
   - Hovering over any diagram element (header field, key container, register, partition) immediately populates the bottom HUD:
     - **Offset / Scope**: e.g., `0x0000..0x001F (32 Bytes)` or `Header Offset + 12`.
     - **Current Value / Format**: e.g., `Magic: 0x46495445 ("FITE")` or `SHA-256 Digest`.
     - **Engineering Rationale**: What code path or hardware register evaluates this field.
3. **Term Flyout Drawers**:
   - Clicking marked technical terms inside the left pane opens a neat side card containing:
     - Plain-language intuition.
     - Concrete reference path or specification clause.
     - 💡 **Why it matters (The engineering trade-off)**.
4. **The Reader's Interactive Controls**:
   - Provide concrete controls (e.g., slider to vary payload size, toggle to simulate invalid signatures or corrupted blocks).
   - Changing the control immediately triggers reactive visual updates on the SVG stage: valid paths illuminate in green, broken states turn rose-red, and the HUD explains the exact line of logic that failed.

---

## 5. Offline-First Deliverable Specifications

- **Format**: Delivered as a single standalone `.html` file.
- **Zero External Dependencies**: Strictly no external CDN references (no Tailwind, Bootstrap, Google Fonts, or external scripts).
- **Universal Local Compatibility**: Launches instantly via `file://` protocol in any standard browser. Uses OS-native high-legibility sans-serif stacks (`-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", sans-serif`).

---

## 6. Pre-Delivery Three-Role Checklist

### Technical Advisor Check
- [ ] **Clarification Loop Executed**: Were ambiguous or missing requirements asked and clarified with the user before building?
- [ ] **Real Engineering Alignment**: Are offsets, data structures, algorithms, and protocols 100% faithful to authentic technical reality?
- [ ] **Physical & Security Boundaries**: Are failure recovery, error paths, and resource boundaries clearly articulated?

### Editor-in-Chief Check
- [ ] **Definite Aha Moment**: Is there an unmistakable, illuminating core insight that anchors the whole explanation?
- [ ] **Pedagogical Translation**: Is the guide written as an engaging, structured masterclass rather than a dry specification dump?
- [ ] **Typography & Comfort**: Is body font size ≥ 14.5px (recommended 15.5–16px) with line-height ≥ 1.8?
- [ ] **Active Reader Variable**: Does the reader have a clear parameter to manipulate and observe cause-and-effect?

### Interactive Director Check
- [ ] **Visual Theme**: Is the canvas clean white/light slate with sharp typography? (Zero dark-neon clutter)
- [ ] **SVG Overflow Prevention (Crucial)**:
  - Are long labels and descriptions broken into multi-line arrays (`lines: [...]`)?
  - Do card heights (`h`) dynamically expand to prevent text running off the bottom border?
- [ ] **Corridor Routing**: Are all arrows routed through clear gutters without slicing through text or cards?
- [ ] **Interactive Responsiveness**: Do tab switches, sliders, hover HUD, and term flyouts work smoothly without console errors?
- [ ] **100% Offline Integrity**: Does the file render and behave perfectly with the network completely disconnected?
