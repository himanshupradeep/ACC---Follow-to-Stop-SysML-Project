# ADAS System Architecture: ACC Follow to Stop (SysML)

This repository contains a comprehensive Model-Based Systems Engineering (MBSE) project detailing the architecture, requirements, and behavior of an **Adaptive Cruise Control (ACC) Follow to Stop** system. 

It demonstrates the application of SysML to map out advanced driver-assistance systems (ADAS) logic, from stakeholder requirements down to ECU-level message exchanges and state logic.

---

## 1. Requirements Diagram
Maps system-level expectations to functional, performance, and safety constraints. It establishes the foundational traceability hierarchy for the entire project.

* **Why we need it:** To visually ensure all stakeholder needs are captured, quantified, and assigned to architectural blocks.
* **When to use it:** Early in the systems engineering lifecycle during the requirements analysis phase.
* **Why we use it:** It creates a single source of truth that links physical design directly back to regulatory and safety goals.
* **What problems it solves:** Prevents "orphan" requirements, unverified constraints, and scope creep by making traceability explicit.

<img width="1227" height="740" alt="image" src="https://github.com/user-attachments/assets/51c46331-5a7b-4ec0-b6b0-4ed7306c5006" />


**SysML Implementation Details:**
* **`«derive»` dependencies:** Used when a sub-requirement is logically or mathematically computed from a parent (e.g., stopping distance physics derived from the follow function).
* **`«refine»` dependencies:** Used to add measurable, testable granularity to a broader requirement (e.g., adding a specific ±0.2s tolerance to a general time gap rule).
* **`«satisfy»` dependencies:** Connects the physical design blocks (like `ACCController`) to the requirements they fulfill.

---

## 2. Block Definition Diagram (BDD)
Defines the static architectural structure of the ACC system, detailing the blocks, their interfaces, and hierarchical decomposition.

* **Why we need it:** To define the system's "bill of materials" and structural boundaries.
* **When to use it:** During system architecture design to map out hardware and logical software blocks.
* **Why we use it:** To clearly distinguish between what the system owns completely and what it borrows from the rest of the vehicle.
* **What problems it solves:** Eliminates ambiguous component ownership and clarifies structural dependencies before any code is written.

<img width="1199" height="797" alt="image" src="https://github.com/user-attachments/assets/52decb40-ae0c-4838-a07c-1b38d665a7df" />


**SysML Implementation Details:**
* **Composition (Filled Diamond):** Used for strict lifecycle ownership. For example, `ACCController` exists solely for the ACC system; if the system is removed, the controller goes with it.
* **Aggregation (Open Diamond):** Used for shared resources. `FrontRadar` is shared with AEB and Lane Keeping, meaning it exists independently of the ACC function.
* **Generalization (Open Triangle):** Establishes inheritance. The `ACCController` inherits base behaviors from an abstract `LongitudinalController`, promoting module reuse across other ADAS features.

---

## 3. Use Case Diagram
Captures the external interactions between actors (Driver, Lead Vehicle, external ECUs) and the system boundary.

* **Why we need it:** To clarify exactly what the system does from an operational perspective.
* **When to use it:** During initial scoping and functional specification.
* **Why we use it:** To map out all triggers and responses without getting bogged down in how the algorithm works.
* **What problems it solves:** Clearly defines the system boundary, ensuring engineers don't over-engineer features outside the scope of the ACC module.

<img width="932" height="792" alt="image" src="https://github.com/user-attachments/assets/675b8188-65c2-43ff-bda7-dbb45129a4d3" />

**SysML Implementation Details:**
* **External Actors:** `Lead Vehicle` is modeled as an actor because it provides direct external stimulus to the system boundary, impacting ACC behavior.
* **`«include»` relationships:** Used for mandatory sub-behaviors. Every time the system executes "Decelerate to Standstill," it *must* execute "Issue Driver Warning."
* **`«extend»` relationships:** Used for conditional behaviors. "Override ACC" only extends the base follow use case at a specific extension point if the pedal actuation condition is met.

---

## 4. Activity Diagram
Models the dynamic control flow and concurrent processing steps of the ACC logic.

* **Why we need it:** To detail the step-by-step algorithm logic and identify parallel processing needs.
* **When to use it:** When defining the functional specification and software architecture.
* **Why we use it:** It bridges the gap between static use cases and actual software implementation.
* **What problems it solves:** Uncovers hidden race conditions and sequential bottlenecks by explicitly modeling which actions must happen simultaneously.

<img width="1229" height="727" alt="image" src="https://github.com/user-attachments/assets/7940acda-e3aa-4f3c-8d2f-c59c345f2bf3" />
<img width="1232" height="719" alt="image" src="https://github.com/user-attachments/assets/c56f074c-41e0-420d-8ae4-912cfe76c1c6" />


**SysML Implementation Details:**
* **Swimlanes:** Partition responsibility. They visually allocate behaviors to physical or logical domains (Driver Input vs. ACC Controller vs. Vehicle Actuation).
* **Fork/Join Nodes:** Explicitly model concurrency. The system uses a Fork to simultaneously monitor ego speed and scan for lead vehicles, using a Join to synchronize that data before computing the follow command.
* **Object Flows:** Differentiate data transfer from control transfer. They show the specific typed data (e.g., `DecelRequest : Acceleration`) passing from one action node to another.

---

## 5. Sequence Diagram
Details the temporal exchange of CAN messages and internal method calls between ECUs during a specific scenario (e.g., Braking to Standstill).

* **Why we need it:** To define protocol timings, lifelines, and the exact order of message transmission.
* **When to use it:** During detailed interface design and network communication planning.
* **Why we use it:** To ensure all distributed ECUs orchestrate their actions in the correct chronological order.
* **What problems it solves:** Prevents timing mismatches, asynchronous communication errors, and CAN bus timeout issues.

<img width="1228" height="776" alt="image" src="https://github.com/user-attachments/assets/a588f98a-7435-4bfb-9022-bd9f58316214" />
<img width="1232" height="566" alt="image" src="https://github.com/user-attachments/assets/95473a42-4916-4bc0-a6b5-40ccaca03c59" />



**SysML Implementation Details:**
* **Synchronous Calls (Solid Arrow):** Used for blocking requests where the sender must wait for processing or confirmation (e.g., a driver button press).
* **Asynchronous Calls (Open Arrow):** Used for "fire-and-forget" network broadcasts (e.g., periodic CAN messages like wheel speed or torque requests).
* **`alt` Combined Fragments:** Used to show mutually exclusive execution paths in a single view. Guard conditions (e.g., `[TTC < 2.5s]` vs `[TTC < 1.0s]`) are dynamically evaluated to route between comfort and emergency braking logic.

---

## 6. State Machine Diagram
Defines the mutually exclusive operational states of the ACC controller and the specific events/guards that trigger transitions.

* **Why we need it:** To map complex, safety-critical modal logic.
* **When to use it:** When defining the core control logic for the ECU software.
* **Why we use it:** To ensure the system always has a deterministic response to any combination of inputs.
* **What problems it solves:** Prevents "state explosion" and undefined system states that cause catastrophic edge-case failures.

<img width="1089" height="796" alt="image" src="https://github.com/user-attachments/assets/c1d4d2d1-ea25-4f3a-9366-1ba92bb8fa42" />


**SysML Implementation Details:**
* **Composite States:** Groups related active states (Speed Control, Follow Control) into one `ACTIVE` state. This allows a single fault transition on the composite boundary to safely handle errors for all sub-states, removing duplicated logic.
* **Orthogonal Regions:** Models independent, concurrent behaviors. Driver pedal monitoring runs safely in an orthogonal region alongside active vehicle control without requiring complex transition webs.
* **History Pseudostate:** Remembers the last active sub-state. If the driver lightly taps the brake and then presses resume, the history state ensures the system returns directly to following the target rather than starting over.
