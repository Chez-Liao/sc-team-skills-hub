---
name: bsp-rdk-code-trace
description: V2
---

name: bsp-rdk-code-trace 
description: Ground every BSP / RDK-B code-trace answer in real DUT console evidence and a verified source path before drawing conclusions. Use whenever investigating boot failures, CCSP/RDK-B component crashes or misbehavior, kernel/u-boot/secure-boot issues, EPON/XGS-PON/DPoE link problems, or any "why does the code do X" question on Broadcom BCM68575-based gateways (Comcast XF10/XER10). Trigger this any time an answer would otherwise be based on how BSP code "usually" behaves rather than on this specific build's console log and this specific board config's compiled code path.
---
 
# BSP / RDK-B Code Trace (Evidence-First)
 
## Why this exists
BSP/RDK-B code paths are config-gated at multiple layers (Kconfig, BitBake
recipe/.bbappend, defconfig, board strap options, secure-boot fuse state).
The same source file can behave completely differently — or not even be
compiled in — depending on which of these is active for the specific unit
under test. Reasoning from "what this code generally does" without checking
what's actually built and what the console actually showed produces
confident-sounding but wrong conclusions. This skill exists to block that
failure mode.
 
## Non-negotiable rule
Do not state a root cause, a code path, or "this is what happens" as a
conclusion unless it is backed by BOTH:
1. An actual DUT console log / command output (boot log, dmesg, CCSP trace,
   WEBPA/PSM dump, etc.) covering the relevant window, AND
2. A code trace performed against the source tree and build config that is
   actually active for that unit (not a generic/mainline/previous-project
   assumption).
 
If either is missing, say so explicitly and ask for it, or clearly label the
answer as an untested hypothesis to verify — never present a guess as a
finding.
 
## Step 1 — Get the evidence
- Ask for (or pull from the repo/ticket) the actual console log covering the
  failure: full boot log from BootROM/OTP stage through kernel, or the
  specific CCSP/RBus/D-Bus trace, or the exact command + output the user ran.
- If only a symptom is described ("wifi agent crashes sometimes"), ask what
  console/log capture exists before speculating on cause.
- Note the exact firmware/build identifier, RDK-B branch, and unit/board
  revision the log came from — a log from a different build is not evidence
  for this one.
 
## Step 2 — Confirm what's actually active for this unit
Before trusting any source file as "the" code path, verify:
- Chipset/board variant (BCM68575 vs other), and which defconfig/board file
  applies
- Yocto MACHINE + which layers/.bbappend are active in this build's
  bblayers.conf
- The actual compiled config, from build artifacts — not by eyeballing a
  `.config` in the tree: `bitbake <recipe> -e | grep ^S=` / `^B=` for
  source/build dirs, and check the applied config-fragment chain in
  .bbappend rather than assuming a top-level defconfig is unmodified
- RDK-B branch/tag/commit (repo manifest) and whether local patches sit on
  top
- The secure-boot stage the log corresponds to (BootROM → OTP fuse check →
  u-boot TPL/SPL → OP-TEE → kernel) — a console string can only be matched
  against the source for the stage that actually printed it
 
## Step 3 — Trace the real source
- Read code from the tree/branch identified in Step 2 — not from memory of
  "how RDK-B usually does this" or a similar past project.
- Follow the actual gating: `#ifdef`/Kconfig guards, syscfg/PSM-driven
  runtime branches, RBus-vs-D-Bus compile-time selection depending on
  version.
- When the trace crosses into Broadcom's HAL layer (`bcm_*`/`hal_*`
  boundary): if the implementation is a closed binary blob, say so
  explicitly — source-level tracing stops there, and behavior at that
  boundary can only be established from console/log evidence, not by
  reading code that isn't available.
- Prefer reading the actual file over grepping for "similar-looking" symbol
  names across unrelated components.
 
## Step 4 — Correlate line by line
- Map each relevant console log line (or call-trace frame, PSM key, TR-181
  parameter) to the exact function/file:line that produced it.
- If a log line can't be matched to any code path found in Step 3, treat
  that as a signal the wrong branch/config/binary was checked — go back to
  Step 2, don't force a match.
 
## Step 5 — State confidence explicitly
Every answer should make clear:
- What was directly confirmed by console log + code trace (cite file:line
  and the matching log line)
- What is inferred but unconfirmed, and what evidence would confirm or rule
  it out
- Where the trace hit a boundary it couldn't cross (closed HAL blob,
  missing log window, unconfirmed build config) — state the gap instead of
  filling it with a guess
 
## RDK-B / CCSP domain pointers
- TR-181 data model definitions live in the component's XML + DML
  implementation; PSM (CCSP Persistent Storage Manager) is where persisted
  parameter values actually come from at runtime — don't assume a hardcoded
  default is what's active.
- CCSP components communicate over D-Bus (older) or RBus (event-driven,
  newer) — confirm which is compiled for this branch before tracing an IPC
  path.
- Per-component trace/log locations and log levels differ by CcspXxxAgent —
  check that component's own logging config rather than assuming a shared
  convention.
- EPON/XGS-PON/DPoE stack issues: correlate optical/link state from console
  against the specific PON MAC/OMCI or DPoE state machine in source —
  link-layer symptoms are frequently misattributed to CCSP/WAN Manager when
  the actual fault is at the PON HAL/driver boundary.
 
## Tooling hooks
- Use the `confluence_rag` MCP to cross-check TR-181/EPON/DPoE spec
  definitions before asserting expected behavior.
- Use Gerrit/git history to check whether the traced code path was recently
  patched — a console log from before/after a patch can look like a
  different code path.
- Cross-check Jenkins build logs to confirm which config fragment/patch set
  was actually included in the build that produced the DUT's console log.
 
## Output format
Structure every answer as:
1. **Evidence used** — console log excerpt / command referenced
2. **Code path traced** — file:line, with the branch/config it was traced
   against
3. **Confidence & gaps** — confirmed vs. inferred vs. unknown
4. **Next verification step**, if inconclusive
 
---
 
## Sources & provenance
 
This skill adapts two public Agent Skills. The write-up above is original —
nothing is copied verbatim from either source. This section records what
was borrowed and how/why it was changed for RDK-B/Broadcom use.
 
**1. intel/linux-kernel-oops**
https://github.com/intel/linux-kernel-oops
(open Agent Skills repo — check the repo's own `COPYRIGHT.md` for exact
terms before redistributing internally)
- *Extracted*: the "evidence-before-conclusion" discipline itself — its
  rule that every call-trace line must be resolved against a real,
  version-matched source tree via symbol lookup before stating a root
  cause, and its staged "What → How → Where" analysis protocol (reflected
  above as Steps 1–5).
- *Not carried over*: its actual tooling — the x86-only `addr2line`/`gdb`
  workflow, the Debian/Fedora/Ubuntu debug-symbol package downloader, and
  the `git clone` against `git.kernel.org` mainline. That tooling assumes
  an x86 target with public upstream kernel history and public distro debug
  symbols, none of which hold for a Broadcom BCM68575 (ARM) vendor kernel
  fork built via a closed Yocto/BitBake SDK.
- *Modification purpose*: broadened scope from "single kernel oops report,
  post-mortem" to "any BSP/RDK-B code-trace question, live or post-mortem";
  replaced the symbol-resolution mechanism with Yocto build-artifact
  verification (Step 2) suited to a closed vendor SDK; added RDK-B/CCSP/
  TR-181/HAL-boundary domain knowledge the source skill has no concept of
  (it's a generic upstream-Linux skill, not RDK-B-aware).
 
**2. hermeticormus/libreembed-claude-code ("Embedded Linux Patterns")**
https://github.com/hermeticormus/libreembed-claude-code
- *Extracted*: one specific pattern — verifying the actually-active kernel
  config via `bitbake <recipe> -e` / the `.bbappend` config-fragment chain,
  instead of trusting a `.config` file directly (their documented
  anti-pattern: direct `.config` edits get silently overwritten by the next
  Yocto build). Reflected above in Step 2.
- *Not carried over*: the rest of that skill — device-tree GPIO/IRQ
  authoring, out-of-tree kernel module Makefile template, sysfs attribute
  pattern, QEMU testing recipe. Those are code-*authoring* patterns for
  writing new drivers/DTS; this skill's job is verifying existing behavior
  against evidence, which is a different task.
- *Modification purpose*: used only as the mechanism inside Step 2; nothing
  else from that skill is in scope here.
 
Check both repos' own license terms before committing derivative text into
an internal-only vs. externally-shared location.
