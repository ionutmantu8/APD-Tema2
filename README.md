# Assignment 2 – APD  
## CHORD Protocol Simulation using MPI

**Student:** Ionut Gabriel Mantu  
**Group:** 333CA  

---

## General Description

This assignment implements a simplified, static version of the **CHORD protocol** using **MPI (Message Passing Interface)**.

The goal is to simulate a **Distributed Hash Table (DHT)** in which nodes are organized in a ring and cooperate to locate the node responsible for a given key.

- Identifier space: **[0, 2⁴ − 1]**
- `m = 4`
- Each MPI process represents a CHORD node
- The system is fully distributed and static (nodes do not join or leave)

---

## Implementation Details

The implementation completes the provided code skeleton, focusing on routing logic and node interaction.  
The solution is structured into **five main components**.

---

## 1. Finger Table Construction (`build_finger_table`)

Each node builds its **Finger Table**, which is used for efficient routing.

Since the ring is static, the table is constructed by iterating through the globally sorted list of node IDs.

For each finger table entry `i` (`0 ≤ i < M`):

1. Compute the start position:
   ```
   start = (self.id + 2^i) % RING_SIZE
   ```
2. Find the successor of `start` using `find_successor_simple`
3. Store the result in:
   ```
   self.finger[i].node
   ```

The finger table enables logarithmic routing by allowing jumps over increasingly large portions of the ring.

---

## 2. Determining the Next Hop (`closest_preceding_finger`)

This function selects the best node to forward a lookup request.

Algorithm:
- Iterate through the finger table from the largest index to the smallest
- Select the first node that lies strictly inside the interval:
  ```
  (self.id, key)
  ```
- Interval checks correctly handle wrap-around cases

Fallback:
- If no suitable finger exists, return `self.successor`

This guarantees progress and avoids infinite routing loops.

---

## 3. Handling Lookup Requests (`handle_lookup_request`)

When a node receives a lookup request:

1. Add the current node ID to the lookup path
2. Check if the key lies in the interval:
   ```
   (self.id, self.successor]
   ```
3. If true:
   - The successor is responsible
   - Add it to the path
   - Send a `TAG_LOOKUP_REP` directly to the initiator
4. Otherwise:
   - Forward the request to the node returned by `closest_preceding_finger`
   - Use `TAG_LOOKUP_REQ`

---

## 4. Sequential Lookup Execution

To ensure deterministic output order, lookups are executed **sequentially**.

- After initialization, only the first lookup request is sent
- When a response is received:
  - The full path is printed
  - The next lookup is automatically launched

Thus, lookup `i + 1` starts only after lookup `i` has completed.

---

## 5. Distributed Service Loop and Termination

Each node runs an infinite loop that processes incoming MPI messages.

### Message Handling
- Uses `MPI_Recv` with:
  - `MPI_ANY_SOURCE`
  - `MPI_ANY_TAG`

### Termination Protocol
- A node sends `TAG_DONE` to all others after completing all its lookups
- Nodes with zero lookups send `TAG_DONE` immediately
- Each node counts received `TAG_DONE` messages

The process terminates when:
```
finished_nodes == world_size
```

---

## Summary

- Fully distributed CHORD simulation
- Static ring topology
- Logarithmic routing using finger tables
- Deterministic output via sequential lookup execution
- Safe global termination without deadlocks
