# Elevator System Design Interview Question

## Problem Statement

Design an **Elevator Control System** for a multi-story building that
efficiently handles elevator requests while minimizing passenger wait
times and travel delays.

The system should:

-   Support multiple elevators operating simultaneously.
-   Accept requests from floors (Up/Down buttons).
-   Accept destination requests from inside elevators.
-   Assign the most appropriate elevator for each request.
-   Handle concurrent requests from multiple users.
-   Be extensible to support features such as maintenance mode, VIP
      service, emergency handling, and future scheduling improvements.

The focus of this design is on creating a clean, object-oriented
architecture.

------------------------------------------------------------------------

# 1. Clarify the Requirements

Before designing the solution, ask questions such as:

-   How many floors and elevators are there?
-   Can multiple users press buttons simultaneously?
-   Do all elevators have the same capacity?
-   Can elevators go out of service?
-   What should be optimized?
    -   Waiting time
    -   Travel time
    -   Energy consumption
    -   Fairness

------------------------------------------------------------------------

# 2. Core Entities

``` text
Building
    ├── Floors
    ├── Elevators
    └── ElevatorController

Elevator
    ├── Current Floor
    ├── Direction
    ├── Requests
    ├── Capacity
    └── Status

HallCall (floor button request)
      ├── Source Floor
      ├── Direction (Up/Down)
      └── Timestamp

CabinRequest (inside elevator request)
      ├── Elevator Id
      ├── Destination Floor
      └── Timestamp
```

------------------------------------------------------------------------

# 3. Main Classes

``` text
Building
- floorCount
- elevators

Elevator
- id
- currentFloor
- direction
- state
- upQueue
- downQueue
- capacity
- currentLoad

HallCall
- fromFloor
- direction

CabinRequest
- elevatorId
- toFloor

ElevatorController
- assignElevator()

Scheduler
- chooseBestElevator()

Floor
- floorNumber
```

------------------------------------------------------------------------

## 3.1 Controller vs Scheduler — Responsibilities

These two classes are intentionally separated by the Single
Responsibility Principle.

### ElevatorController — "What needs to happen?"

-   The **entry point** for all external events: hall calls, cabin
    requests, door signals, and emergency alerts.
-   Owns the fleet: knows how many elevators exist, their current
    states, and routes events to the right place.
-   Delegates the *which elevator is best?* decision to the Scheduler.
-   Enforces policies: maintenance lock, emergency mode, VIP override.

Think of it as the **building's control panel** — it receives signals
and coordinates the response.

### Scheduler — "Which elevator is best?"

-   A pure decision engine: given a request and a snapshot of all
    elevator states, it returns the best eligible elevator.
-   Has no knowledge of policies, emergency modes, or building
    structure.
-   Only concerns itself with eligibility filtering, scoring, and
    optimization logic.
-   Easily swappable — a cost-based scheduler can be replaced with an
    ML-based one without touching the controller.

Think of it as the **algorithm brain** — stateless, focused purely on
the assignment decision.

### Interaction Flow

``` text
HallCall arrives
      |
ElevatorController        <- owns policy and routing
      |
  Scheduler.assign()      <- owns the "which elevator?" decision
      |
Best elevator returned
      |
ElevatorController adds
request to elevator queue  <- back to controller to execute
```

### Responsibilities at a Glance

``` text
Concern                        | Controller | Scheduler
-------------------------------|------------|----------
Knows about all elevators      | Yes        | Yes (read-only)
Enforces business rules        | Yes        | No
Chooses best elevator          | No         | Yes
Swappable algorithm            | No         | Yes
Handles emergencies            | Yes        | No
```

------------------------------------------------------------------------

# 4. Elevator States

-   Idle
-   Moving Up
-   Moving Down
-   Door Opening
-   Door Open
-   Door Closing
-   Maintenance

------------------------------------------------------------------------

# 5. Request Lifecycle

``` text
User presses button
      ↓
Request created
      ↓
Elevator Controller
      ↓
Scheduler
      ↓
Best elevator selected
      ↓
Request added to queue
      ↓
Elevator moves
      ↓
Passenger boards
      ↓
Destination served
```

------------------------------------------------------------------------

# 6. Elevator Allocation Strategy

Avoid always selecting the nearest elevator.

Use a two-phase selection process:

1. Eligibility filter (hard constraints)
- Exclude elevators in maintenance/out-of-service state.
- Exclude elevators that are at full capacity.
- Exclude elevators that cannot accept the request due to policy
      constraints (for example, reserved or emergency lock).

2. Scoring (soft optimization)

Consider:

-   Distance from request
-   Current direction
-   Existing queued stops
-   Current load
-   Elevator availability

Example scoring:

``` text
score =
(w1 * distance)
+ (w2 * direction mismatch penalty)
+ (w3 * queued stops penalty)
+ (w4 * load factor penalty)
```

**What are tunable weight coefficients?**

-   **Coefficient** — a number multiplied against a factor to scale its
    effect. In `w1 * distance`, `w1` is the coefficient: it stretches
    or shrinks how much distance contributes to the final score.
-   **Weight** — a coefficient used to express *relative importance*.
    If `w1 = 10` and `w3 = 1`, distance is 10× more important than
    queued stops.
-   **Tunable** — the values are not hardcoded. They are stored in
    configuration so an operator can adjust them per deployment without
    changing the algorithm itself.

In short: numbers that control how much each factor matters, adjustable
without touching the code.

**Weight definitions:**

| Weight | Factor                  | What it penalizes                                              | Typical priority |
|--------|-------------------------|----------------------------------------------------------------|------------------|
| `w1`   | Distance                | Floors the elevator must travel to reach the requester         | High             |
| `w2`   | Direction mismatch      | Elevator moving away from or opposite to the request direction | Medium-high      |
| `w3`   | Queued stops            | Number of stops already committed before reaching requester    | Medium           |
| `w4`   | Load factor             | How full the elevator is (currentLoad / capacity)              | Low-medium       |

Weights are tunable per deployment — not hardcoded constants:

-   **Peak morning rush** (everyone going up): raise `w2` to strongly
    prefer elevators already heading up.
-   **High-capacity building**: raise `w4` to avoid assigning
    near-full cars.
-   **Small building**: raise `w1` to prioritize proximity above all.
-   **VIP floor**: lower all weights for a reserved elevator so it
    always wins for that floor.

**Analogy — think of weights like seasoning a dish:**

| Ingredient         | Weight analogy                                      |
|--------------------|-----------------------------------------------------|
| Salt (distance)    | Added generously — it matters most                  |
| Pepper (direction) | A fair amount — noticeable but secondary            |
| Chilli (stops)     | A little — affects the result but subtly            |
| Sugar (load)       | Just a pinch — rarely the deciding factor           |

The recipe (algorithm) stays the same. You adjust the amounts (weights)
per taste (building scenario).

Choose the elevator with the lowest score.

------------------------------------------------------------------------

# 7. Request Queues

Maintain separate queues for upward and downward movement.

``` text
Up Queue
3
5
7
11

Down Queue
18
15
12
9
```

Primarily serve requests in the current direction (SCAN-like behavior),
but add anti-starvation rules for fairness:

-   Max-wait threshold: if an opposite-direction request exceeds a wait
      limit, allow earlier reversal or reassignment.
-   Aging boost: increase priority of older requests over time.
-   Global rebalance: idle elevators can be reassigned to long-wait
      floors.

------------------------------------------------------------------------

# 8. Scheduler Flow

Use a hybrid efficient scheduling algorithm:

-   Global dispatch: cost-based nearest-car selection (lowest score
    after eligibility filtering).
-   Per-elevator serving: LOOK/SCAN-style directional queue processing
    with anti-starvation overrides.

``` text
Algorithm: Hybrid Cost + LOOK

onHallCall(request):
  candidates = elevators where isEligible(elevator, request)
  for each elevator in candidates:
    score[elevator] =
      w1 * distanceToPickup
    + w2 * directionMismatchPenalty
    + w3 * queuedStopsPenalty
    + w4 * loadFactorPenalty
    - w5 * waitingTimeAgingBoost
  best = elevator with minimum score
  assign request to best

onElevatorStep(elevator):
  serve stops in current direction first (LOOK)
  if opposite queue has starved request (wait > threshold):
    allow early reversal or reassignment
```

Time complexity per new hall call:

-   Candidate evaluation and scoring: **O(number of elevators)**
-   Queue insertion for selected elevator: **O(log stops)** with a
    sorted set / heap (or **O(stops)** with a list)

Typical dispatch complexity: **O(E + log S)**, where **E** = elevators,
**S** = stops in the selected elevator queue.

------------------------------------------------------------------------

# 9. Handling New Requests

If an elevator is already moving in the same direction as a new
compatible request, insert the stop in sorted order instead of
appending it.

If the new request is in the opposite direction, place it in the
opposite-direction queue and serve it according to fairness rules.

------------------------------------------------------------------------

# 10. Edge Cases

-   Elevator full
-   Elevator under maintenance
-   All elevators busy
-   Duplicate requests from the same floor
-   Passenger changes destination
-   Emergency mode
-   VIP/reserved elevator mode

------------------------------------------------------------------------

# 11. Concurrency

A production system should support concurrent events.

``` text
Controller Thread
        ↓
Thread-safe Request Queue
        ↓
Scheduler Thread
        ↓
Individual Elevator Threads
```

Each elevator processes its own queue independently.

------------------------------------------------------------------------

# 12. Sample Interfaces

``` java
interface Scheduler {
      Elevator assign(HallCall request);
}
```

``` java
interface Elevator {
    void move();
      void addCabinRequest(CabinRequest request);
      void addHallStop(int floor);
    void step();
}
```

------------------------------------------------------------------------

# 13. High-Level Architecture

``` text
             Building
                 │
      Elevator Controller
                 │
          Scheduler Logic
      ┌──────────┼──────────┐
      │          │          │
 Elevator1  Elevator2  Elevator3
      │          │          │
 RequestQ   RequestQ   RequestQ
```

------------------------------------------------------------------------

# 14. Possible Enhancements

-   AI-based predictive dispatch
-   Destination control system
-   Energy-saving mode
-   Express elevators
-   Dynamic zoning
-   Fault tolerance and automatic reassignment
-   Monitoring and analytics dashboard

------------------------------------------------------------------------

# What Interviewers Evaluate

A strong solution demonstrates:

-   Requirement clarification
-   Good object-oriented design
-   Separation of responsibilities
-   Efficient scheduling algorithm
-   Scalability
-   Thread safety
-   Edge-case handling
-   Ability to discuss trade-offs and future improvements
