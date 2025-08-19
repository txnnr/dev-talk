## Building Feedback Systems with OT and CRDTs


## What are Real-Time Streaming Apps?

- Applications where **state updates flow between users instantly**
- Examples: Google Docs, Figma, Miro, Notion comments
- Behave **offline-first**, but **sync-later**, without conflicts
- Core challenges:
  - Concurrent edits
  - Unstable networks
  - Merge conflicts

---

## Two approaches to solve this

### 1. **Operational Transform (OT)** — *Google Docs era*
### 2. **CRDT (Conflict-Free Replicated Data Types)** — *Modern, offline-first sync*

---

## Operational Transform (OT)

- Clients send **operations**: e.g. "insert at position 5"
- Server performs **transform functions** to reorder/rescale
- Goal: *All clients apply the same ops in the same state*

```

Initial: "hello"

User A → Insert(1,"X")
User B → Insert(3,"Y")

Server:
apply B → "helYlo"
transform A relative → "hXelYlo"

```

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

> **Data structures that always converge** — regardless of the order ops arrive.

- Each edit carries enough metadata (timestamps / causal info)
- Merge is **mathematical** → *no transforms needed*
- Perfect for **offline-first + P2P** syncing

---

### CRDT Merge Example

```

"a"(A1) "b"(A2) "c"(A3)

User A inserts X after A1
User B inserts Y after A2

IDs carry causal links

Merge order doesn't matter
→ Converges to   a X b Y c

```

---

## Types of CRDT

| Type              | Used For                  |
|-------------------|---------------------------|
| Sequence CRDT     | text streams              |
| JSON/Map CRDT     | docs / objects            |
| Grow-only / OR-Set | basic sets & lists        |

Popular libraries:
- **Automerge** → JSON CRDT (local-first, JS/Rust)
- **Y.js** → text/array/map CRDT (web-scale)
- **Fluid Framework** → enterprise Microsoft Loop

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

---

## How Do We Implement Real-Time Streaming?

You can pair OT/CRDT models with different networking layers:

---

### 🔌 WebSockets (Client <-> Server)

> A persistent **TCP connection** between browser & server.

- Server forwards messages between clients
- Perfect for **centralized OT systems** (ShareDB, Fluid)
- Also works for CRDT when you want to avoid P2P networking

✅ Easier NAT traversal  
✅ Simple to scale horizontally  
❌ Server becomes a sync bottleneck / single point

---

**WebSocket stack looks like:**

```

Browser <── WebSocket ──> Server <──> Other Browsers

```

---

### 📡 WebRTC (Peer <-> Peer)

> Browser-to-browser connection designed for realtime video/data.

Used by CRDT libraries like **Automerge Repo**, **Yjs**, **Liveblocks**, etc.

**Requires three pieces:**

| Acronym | Stands For       | Purpose                     |
|--------|------------------|-----------------------------|
| STUN   | Session Traversal Utilities for NAT | Discovers your public IP/port (NAT traversal) |
| TURN   | Traversal Using Relays around NAT  | Relays traffic *if* direct P2P fails |
| ICE    | Interactive Connectivity Establishment | Chooses the best route between peers using candidates from STUN/TURN |

---

**WebRTC stack looks like:**

```

Browser 1 <--(signaling WS)--> Server <--(signaling WS)--> Browser 2
│
ICE: tries P2P via STUN/TURN
│
└──> Direct peer connection established

```

✅ True peer-to-peer  
✅ Works offline → sync later  
❌ Requires a signaling channel (usually a WebSocket)  
❌ Relies on STUN/TURN infrastructure for NAT punching  
❌ Slightly more complex flow

---

### 🛠 Practical library examples

| Sync Model | Transport | Library           |
|------------|-----------|-------------------|
| OT         | WebSocket | ShareDB           |
| CRDT       | WebSocket | Automerge Repo    |
| CRDT       | WebRTC    | Yjs, Liveblocks   |
```

---



### 🛠 Practical Stack Examples

| Sync Model | Transport | Library        |
|------------|-----------|----------------|
| OT         | WebSocket | ShareDB        |
| CRDT       | WebRTC    | Y.js, Automerge|
| CRDT       | WebSocket | Automerge, Liveblocks |


---

## Real-World Usage

| Use Case                  | Best Choice     |
|---------------------------|-----------------|
| Google-style doc editing  | OT / Y.js       |
| Whiteboard / comments     | CRDT (Automerge)|
| Enterprise multi-user app | Fluid Framework |
| Peer-to-peer clipboard    | CRDT            |

---


## Summary

- Real-time streaming = **Instant UI + async sync**
- OT is reliable but server-centric
- CRDT unlocks true **offline-first developer UX**
- With tools like Automerge & Y.js, we can build this *today* in **React + TypeScript**
