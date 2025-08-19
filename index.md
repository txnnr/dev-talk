---
layout: default
description: Tanner - Dev Talk Aug 19/2025
---

## Building Feedback Systems with OT and CRDTs

## What are Real-Time Streaming Apps?

* Applications where **state updates flow between users instantly**
* Examples: Google Docs, Figma, Miro, Notion comments
* Behave **offline-first**, but **sync-later**, without conflicts
* Core challenges:

  * Concurrent edits
  * Unstable networks
  * Merge conflicts

---

## Two Approaches to Solve This

### 1. **Operational Transform (OT)** — *Google Docs era*

### 2. **CRDT (Conflict-Free Replicated Data Types)** — *Modern, offline-first sync*

---

## Operational Transform (OT)

* Clients send **operations** (e.g. "insert at position 5")
* When operations arrive **out of order**, the server applies a *transform function* → adjusts an incoming op so it still makes sense after previous edits
* Goal: *All clients apply operations in the **same resulting order** so they converge*

**Example:**

Initial string: `"hello"`

```
User A → Insert(1, "X")
User B → Insert(3, "Y")
```

Operations arrive at the server in the order: **B**, then **A**

```
apply opB → "helYlo"
transform opA to account for opB
(since B inserted a char at position 3, A’s original "insert at 1" still valid)
apply transformed-A → "hXelYlo"
```

> 🔧 *The "transform" step is a tiny function that modifies position values so the intent of each operation is preserved even when edits happen concurrently.*

---

### OT Strengths

✅ Proven (used in Google Docs, Quill)
✅ Small payloads
✅ Great for strict ordering (code, rich-text)

### OT Drawbacks

❌ Usually requires **central server**
❌ Hard to support **offline editing**
❌ Complex transforms for nested JSON/trees → edge-case nightmares

---

## What are CRDTs?

> **Conflict-Free Replicated Data Types** — data structures that
> **automatically merge** concurrent edits *without needing transforms or a central server*.

* Each operation carries just enough metadata (unique IDs, causal links)
* Replicas exchange operations in **any order**
* Merge logic guarantees that all replicas **eventually converge**

**Example (Sequence CRDT):**

Initial:

```
"h"(A1), "e"(A2), "l"(A3), "l"(A4), "o"(A5)
```

Two users edit concurrently:

```
User A → insert "X" after A1
User B → insert "Y" after A3
```

CRDT merges based on positions **between IDs**, not absolute indexes → order of arrival doesn’t matter:

```
"h"(A1), "X", "e"(A2), "l"(A3), "Y", "l"(A4), "o"(A5)
→ hXelYlo
```

> 🧠 *No transform logic required — the merge strategy is built into the CRDT data structure itself.*

---

## Types of CRDT

| Type               | Used For           |
| ------------------ | ------------------ |
| Sequence CRDT      | text streams       |
| JSON/Map CRDT      | docs / objects     |
| Grow-only / OR-Set | basic sets & lists |

Popular libraries:

* **Automerge** → JSON CRDT (local-first, JS/Rust)
* **Y.js** → text/array/map CRDT (web-scale)
* **Fluid Framework** → enterprise Microsoft Loop

---

## Why CRDTs are exciting

✅ Work *offline*: all edits are local
✅ Peer-to-peer capable (WebRTC)
✅ Easy for **React apps** (local state + sync)

---

## But… CRDTs are not perfect

❌ Metadata can grow → memory & WS payloads
❌ Hard to garbage-collect tombstones
❌ Unstable networks → constant reconnect/retry logic needed
❌ Sync algorithms still evolving

---

## How Do We Implement Real-Time Streaming?

Once we pick **OT** or **CRDT** as our *conflict-resolution model*, we still need a way for nodes to actually **talk to each other**.
This is where the **network transport** comes in.

---

### 🔌 WebSockets — *Client ↔ Server*

> Long-lived TCP channel between browser and central server.

* All updates flow to/from the server
* Easier to scale, works everywhere (firewalls/NATs)
* Common in OT setups like *ShareDB* — but CRDTs like *Automerge Repo* can also use it

✅ Simple deployment
✅ Works even on corporate networks
❌ Server becomes a bottleneck / single point

Diagram:

```
Browser 1 ←→ WebSocket ←→ Server ←→ WebSocket ←→ Browser 2
```

---

### 📡 WebRTC — *Peer ↔ Peer*

> Browser-to-browser realtime channel.

* Great for *offline-first CRDT syncing*
* Clients attempt to connect directly — if that fails, fall back to a relay
* Requires a *signaling* channel (often still a WebSocket!) to get started

**Needs three pieces of tech:**

| Acronym  | Purpose                                             |
| -------- | --------------------------------------------------- |
| **STUN** | Discover your public IP/port (NAT traversal)        |
| **TURN** | Relay server used *if direct connection fails*      |
| **ICE**  | Algorithm that tries all candidates & picks a route |

✅ True peer-to-peer
✅ Works offline → sync when reconnected
❌ Harder to debug / NAT punch
❌ Still needs signaling infrastructure

Diagram:

```
Browser 1 ←─(signaling over WS)─→ Server ←─(signaling over WS)─→ Browser 2
↓ ICE (STUN/TURN discovery)
Attempts direct P2P connection
```

---

### 🛠️ Practical Library Examples

| Model | Transport | Examples         |
| ----- | --------- | ---------------- |
| OT    | WebSocket | ShareDB          |
| CRDT  | WebSocket | Automerge Repo   |
| CRDT  | WebRTC    | Y.js, Liveblocks |

---

### 🛠 Practical Stack Examples

| Sync Model | Transport | Library               |
| ---------- | --------- | --------------------- |
| OT         | WebSocket | ShareDB               |
| CRDT       | WebRTC    | Y.js, Automerge       |
| CRDT       | WebSocket | Automerge, Liveblocks |

---

## Real-World Usage

| Use Case                  | Best Choice      |
| ------------------------- | ---------------- |
| Google-style doc editing  | OT / Y.js        |
| Whiteboard / comments     | CRDT (Automerge) |
| Enterprise multi-user app | Fluid Framework  |
| Peer-to-peer clipboard    | CRDT             |

---

## Summary

* Real-time streaming = **Instant UI + async sync**
* OT is reliable but server-centric
* CRDT unlocks true **offline-first developer UX**
* With tools like Automerge & Y.js, we can build this *today* in **React + TypeScript**

---

## Sources / Further Reading

* [Fluid Framework](https://fluidframework.com/)
* [Azure Fluid Relay](https://azure.microsoft.com/en-us/products/fluid-relay/#overview)
* [Automerge GitHub](https://github.com/automerge/automerge)
* [Local-First Software Essay (Ink & Switch)](https://www.inkandswitch.com/essay/local-first/)
* [CRDT.tech](https://crdt.tech/)
* [Real-Time Collaboration: OT vs CRDT (Tiny)](https://www.tiny.cloud/blog/real-time-collaboration-ot-vs-crdt/)
