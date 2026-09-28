# GasGrid — LPG Supply Network & Resource Allocation System

GasGrid is an **Operating System and Database Management System based LPG supply and resource allocation system** designed to simulate and optimize the distribution of LPG under limited resources and competing delivery requests.

Unlike a conventional LPG booking website, GasGrid treats each LPG request as a **process** and models cylinders, vehicles, delivery agents, warehouse capacity, and delivery slots as **system resources**. The system applies Operating System concepts such as **process scheduling, synchronization, resource allocation, deadlock detection, starvation prevention, and aging**, while Oracle DBMS handles **transactions, constraints, PL/SQL, triggers, auditing, analytics, and inventory management**.

The project aims to demonstrate how OS and DBMS concepts can work together to solve a practical resource-constrained supply problem.

---

## Problem Statement - increase

LPG distribution involves multiple customers and delivery requests competing for limited resources such as:

* LPG cylinders
* Warehouse inventory
* Delivery vehicles
* Delivery agents
* Delivery slots
* Warehouse capacity

During normal demand, high demand, emergencies, or limited inventory situations, simply storing bookings is not enough.

The system must decide:

* Which request should be processed first?
* Which resources should be allocated?
* How should emergency requests be prioritized?
* What happens when multiple requests require the same cylinder or vehicle?
* How can deadlock be detected and recovered?
* How can starvation of normal requests be prevented?
* How can inventory remain consistent during concurrent bookings?
* When should additional LPG stock be ordered?

GasGrid addresses these problems using a combination of **Operating System algorithms and DBMS mechanisms**.

---

## Key Idea

GasGrid models the LPG distribution system as a **resource-constrained operating environment**.

```text
LPG Request      →      Process
Cylinder         →      Resource
Vehicle          →      Resource
Delivery Agent   →      Resource
Warehouse Space  →      Resource
Emergency Order  →      High-Priority Process
Booking Queue    →      Ready Queue
Processing       →      CPU/Scheduler Simulation
Resource Conflict →     Synchronization
Circular Waiting →      Deadlock
Long Waiting     →      Starvation
Priority Increase →     Aging
```

This allows theoretical OS concepts to be applied to a realistic LPG distribution scenario.

---

## Main Features

### 1. LPG Type & Cylinder Management

The system distinguishes between different LPG categories and cylinder types.

Example:

```text
LPG Type
│
├── Domestic LPG
│   └── Domestic Cylinder – 14.2 kg
│
└── Commercial LPG
    └── Commercial Cylinder – 19 kg
```

The system maintains stock and allocation separately according to cylinder type.

---

### 2. LPG Request Management

Each customer request is converted into a system process.

A request contains information such as:

* Process ID
* Customer ID
* LPG type
* Cylinder type
* Request priority
* Arrival time
* Processing time
* Required resources
* Process state
* Start time
* Completion time
* Waiting time
* Turnaround time

Process states include:

```text
NEW
 ↓
READY
 ↓
RUNNING
 ↓
WAITING / BLOCKED
 ↓
COMPLETED
```

---

## Operating System Module

The OS module is implemented in **C**.

### Process Management

Every LPG delivery request is represented as a process.

The system maintains:

* Process ID
* Arrival time
* Burst/processing time
* Priority
* State
* Required resources
* Waiting time
* Turnaround time
* Completion time

---

### Process Scheduling

GasGrid supports multiple scheduling strategies for competing LPG requests:

* First Come First Serve (FCFS)
* Shortest Job First (SJF)
* Shortest Remaining Time First (SRTF)
* Priority Scheduling
* Round Robin
* Multilevel Feedback Queue (MLFQ)

The scheduler can compare algorithms using:

* Waiting Time
* Turnaround Time
* Response Time
* Completion Time
* Average Waiting Time
* Average Turnaround Time

A Gantt chart can be generated to visualize scheduling decisions.

---

### Priority Scheduling

Different LPG requests can have different priorities.

Example:

```text
Emergency / Critical Request   → Highest
Commercial Urgent Request      → High
Normal Domestic Request        → Medium
Scheduled Request              → Low
```

Priority can also be dynamically modified based on waiting time.

---

### Preemption

An emergency request can interrupt a lower-priority request when the system allows preemption.

Example:

```text
Normal Request P1
      ↓
   RUNNING
      ↓
Emergency Request P2 arrives
      ↓
P1 → WAITING
P2 → RUNNING
```

After the emergency request is completed, P1 can resume.

---

### Synchronization

Multiple LPG requests may attempt to access the same limited resource simultaneously.

Example:

```text
Available cylinders = 1

P1 → requests cylinder
P2 → requests cylinder

        ↓
 Critical Section
        ↓

Only one process can reserve
the cylinder at a time.
```

Synchronization prevents:

* Duplicate allocation
* Negative inventory
* Race conditions
* Inconsistent resource states

Concepts used include:

* Mutex
* Semaphore
* Critical Section

---

### Resource Allocation

A single LPG request may require multiple resources.

Example:

```text
Request P1

Required:
├── 1 LPG Cylinder
├── 1 Vehicle
├── 1 Delivery Agent
└── 1 Delivery Slot
```

The resource manager checks availability before granting resources.

---

### Banker's Algorithm

Before granting a resource request, GasGrid can check whether the allocation keeps the system in a **safe state**.

The system maintains:

```text
Available
Maximum
Allocation
Need
```

The Banker’s Algorithm is used to avoid unsafe resource allocation.

---

### Deadlock Detection

GasGrid simulates real resource deadlock scenarios.

Example:

```text
P1 holds Vehicle V1
P1 waits for Cylinder C1

P2 holds Cylinder C1
P2 waits for Vehicle V1
```

This produces:

```text
P1 → C1
↑    ↓
V1 ← P2
```

The system detects the circular wait and identifies the deadlock.

---

### Deadlock Recovery

After detecting deadlock, the system can:

1. Identify a victim process
2. Release its resources
3. Roll back the affected operation
4. Reallocate resources
5. Resume waiting processes

---

### Starvation and Aging

Continuous emergency requests may cause normal requests to wait indefinitely.

GasGrid addresses this using **aging**.

```text
Long waiting time
       ↓
Priority increases
       ↓
Request gets scheduled
```

This helps maintain fairness while still supporting priority-based emergency handling.

---

# DBMS Module

The database component uses **Oracle Database**.

The DBMS is responsible for maintaining persistent system information and ensuring data consistency during resource allocation and delivery operations.

---

## Database Responsibilities

The database manages:

* Customers
* LPG types
* Cylinder types
* Cylinders
* Inventory
* Warehouses
* Vehicles
* Delivery agents
* LPG requests
* Processes
* Resource allocations
* Deliveries
* Payments
* Inventory transactions
* Audit records
* Alerts

---

## Inventory Management

GasGrid maintains more than a simple stock count.

Inventory transactions are recorded using a ledger.

Example transaction types:

```text
IN
RESERVED
RELEASED
DELIVERED
RETURNED
DAMAGED
ADJUSTED
```

This makes it possible to track how inventory changed over time.

Example:

```text
100 cylinders
      ↓
10 reserved
      ↓
5 delivered
      ↓
2 returned
      ↓
Current inventory calculated
```

---

## Database Transactions

A booking operation involves multiple database operations.

Example:

```text
START TRANSACTION
      ↓
Validate Customer
      ↓
Validate LPG Type
      ↓
Check Inventory
      ↓
Reserve Cylinder
      ↓
Allocate Resources
      ↓
Create Delivery
      ↓
Record Inventory Transaction
      ↓
COMMIT
```

If an operation fails:

```text
ROLLBACK
```

This prevents partial updates and maintains database consistency.

---

## ACID Properties

GasGrid demonstrates the use of:

* **Atomicity**
* **Consistency**
* **Isolation**
* **Durability**

These properties are particularly important during concurrent LPG bookings and inventory allocation.

---

## PL/SQL

Oracle PL/SQL is used to implement business logic inside the database.

### Stored Procedures

Examples:

```text
CREATE_BOOKING
RESERVE_INVENTORY
ALLOCATE_RESOURCE
ASSIGN_VEHICLE
COMPLETE_DELIVERY
CANCEL_BOOKING
PROCESS_EMERGENCY_REQUEST
```

### Functions

Examples:

```text
CALCULATE_PRIORITY_SCORE
CHECK_INVENTORY
CALCULATE_WAITING_TIME
CALCULATE_UTILIZATION
PREDICT_STOCK_STATUS
```

---

## Triggers

Triggers are used to automatically respond to important database events.

Examples:

```text
Low Inventory
      ↓
Trigger
      ↓
Create Stock Alert
```

Other possible triggers:

* Booking cancellation → release reserved inventory
* Delivery completion → update inventory
* Inventory update → create audit record
* Important data modification → maintain audit log

---

## Advanced SQL & Analytics

The project uses advanced SQL concepts including:

* INNER JOIN
* LEFT JOIN
* Subqueries
* Correlated subqueries
* Common Table Expressions (CTEs)
* Aggregate functions
* CASE expressions
* Window functions
* Ranking
* Running totals
* Historical analysis

Example analytical questions:

* Which LPG type has the highest demand?
* Which warehouse has the highest utilization?
* Which warehouse is likely to run out first?
* Which delivery agent has the highest completion rate?
* What is the average request waiting time?
* Which requests experienced the highest delay?
* Which resources are underutilized?
* When should inventory be reordered?

---

# OS + DBMS Integration

The major objective of GasGrid is not to keep the OS and DBMS modules separate.

They work together.

```text
                    CUSTOMER
                       │
                       ▼
                LPG REQUEST
                       │
                       ▼
                 FLASK BACKEND
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
       ORACLE DBMS           OS ENGINE
             │                   │
       Validate Data        Create Process
             │                   │
             └─────────┬─────────┘
                       ▼
                  SCHEDULER
                       │
                       ▼
              RESOURCE MANAGER
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
         Resources            Resources
        Available             Unavailable
             │                   │
             ▼                   ▼
         Allocate          WAIT / BLOCK
             │
             ▼
       Oracle Transaction
             │
             ▼
       Delivery Processing
             │
             ▼
       Inventory Update
             │
             ▼
           COMMIT
```

---

## Practical Integration Examples

### Scheduling + Database

LPG requests are stored in Oracle.

The OS engine retrieves ready requests and applies the selected scheduling algorithm.

The scheduling result is then stored back in the database.

---

### Synchronization + Transactions

Two requests attempting to reserve the last available cylinder are synchronized.

The database transaction ensures that only one request successfully reserves the resource.

---

### Resource Allocation + Inventory

The OS resource manager checks resource availability while Oracle maintains the actual inventory and allocation records.

---

### Analytics + Resource Planning

Historical database information can be analyzed to estimate future demand.

The result can influence:

* Resource allocation
* Scheduling priority
* Inventory planning
* Reorder alerts

---

# Technology Stack

| Component             | Technology            |
| --------------------- | --------------------- |
| OS Simulation         | C                     |
| Database              | Oracle Database       |
| Database Programming  | SQL, PL/SQL           |
| Backend               | Python Flask          |
| Frontend              | HTML, CSS, JavaScript |
| Database Connectivity | Python Oracle Driver  |
| Version Control       | Git & GitHub          |

---

# Project Architecture

```text
                    ┌─────────────────────┐
                    │      Frontend       │
                    │ HTML / CSS / JS     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    Flask Backend    │
                    └──────────┬──────────┘
                               │
                 ┌─────────────┴─────────────┐
                 ▼                           ▼
       ┌──────────────────┐        ┌──────────────────┐
       │   OS Engine      │        │   Oracle DBMS    │
       │                  │        │                  │
       │ Process Mgmt     │        │ Tables           │
       │ Scheduling       │◄──────►│ Transactions     │
       │ Synchronization  │        │ PL/SQL           │
       │ Resource Mgmt    │        │ Triggers         │
       │ Deadlock         │        │ Analytics        │
       │ Starvation       │        │ Audit            │
       └──────────────────┘        └──────────────────┘
```

---

# Repository Structure

```text
GasGrid/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── docs/
│   ├── project-proposal.md
│   ├── architecture.md
│   ├── os-design.md
│   ├── dbms-design.md
│   ├── integration.md
│   ├── algorithms.md
│   ├── testing.md
│   │
│   └── diagrams/
│       ├── system-architecture.png
│       ├── process-state.png
│       ├── er-diagram.png
│       ├── dfd-level-0.png
│       ├── dfd-level-1.png
│       ├── resource-allocation.png
│       ├── deadlock.png
│       └── sequence-diagram.png
│
├── os_engine/
│   ├── include/
│   ├── src/
│   │
│   ├── process/
│   ├── scheduling/
│   ├── synchronization/
│   ├── resource/
│   ├── deadlock/
│   ├── starvation/
│   ├── simulation/
│   └── tests/
│
├── database/
│   ├── 01_schema/
│   ├── 02_data/
│   ├── 03_queries/
│   ├── 04_views/
│   ├── 05_plsql/
│   ├── 06_triggers/
│   ├── 07_transactions/
│   ├── 08_indexes/
│   ├── 09_audit/
│   └── 10_analytics/
│
├── backend/
│   ├── app.py
│   ├── config.py
│   ├── routes/
│   ├── services/
│   ├── database/
│   └── os_bridge/
│
├── frontend/
│   ├── index.html
│   ├── dashboard.html
│   ├── booking.html
│   ├── inventory.html
│   ├── scheduling.html
│   ├── resources.html
│   ├── deliveries.html
│   └── analytics.html
│
├── scenarios/
│   ├── normal_demand/
│   ├── high_demand/
│   ├── emergency_request/
│   ├── concurrent_booking/
│   ├── deadlock/
│   ├── starvation/
│   ├── insufficient_inventory/
│   └── transaction_failure/
│
├── tests/
│   ├── os_tests/
│   ├── dbms_tests/
│   └── integration_tests/
│
└── screenshots/
    ├── dashboard/
    ├── os/
    └── dbms/
```

---

# Important Test Scenarios

GasGrid will be evaluated using realistic scenarios.

### Scenario 1 — Normal Demand

Multiple normal LPG requests arrive and are scheduled using the selected scheduling algorithm.

### Scenario 2 — High Demand

A large number of requests compete for limited cylinders, vehicles, and delivery agents.

### Scenario 3 — Emergency Request

An emergency request arrives while normal requests are being processed.

The scheduler gives it higher priority.

### Scenario 4 — Concurrent Booking

Two requests attempt to reserve the last available LPG cylinder simultaneously.

Synchronization and database transactions prevent inconsistent allocation.

### Scenario 5 — Deadlock

Two processes hold resources required by each other.

The system detects the deadlock and performs recovery.

### Scenario 6 — Starvation

Continuous high-priority requests cause a normal request to wait.

Aging gradually increases its priority.

### Scenario 7 — Insufficient Inventory

Demand exceeds available stock.

The system blocks or delays requests and generates an inventory alert.

### Scenario 8 — Transaction Failure

A booking operation fails after partial processing.

The transaction is rolled back to maintain database consistency.

---

# Dashboard

The GasGrid dashboard will provide a unified view of the system.

Expected information includes:

* Active LPG requests
* Ready queue
* Waiting processes
* Blocked processes
* Emergency requests
* Available cylinders
* Reserved cylinders
* Vehicle availability
* Delivery agent availability
* Warehouse utilization
* Current scheduling algorithm
* Gantt chart
* Resource allocation status
* Deadlock status
* Starvation alerts
* Low inventory alerts
* Demand analytics
* Delivery performance

---

# Expected Outcomes

The project aims to demonstrate:

1. Practical application of Operating System concepts.
2. Practical application of advanced DBMS concepts.
3. Integration between OS resource management and database transactions.
4. Safe handling of concurrent LPG requests.
5. Efficient scheduling of competing requests.
6. Detection and recovery from resource deadlocks.
7. Prevention of starvation using aging.
8. Consistent inventory management.
9. Analytical decision-making using historical data.
10. A realistic simulation of an LPG supply network.

---

# Future Scope

The system can be extended with:

* Real-time vehicle tracking
* GPS-based route optimization
* Demand forecasting using Machine Learning
* Dynamic pricing
* Multi-warehouse optimization
* IoT-based cylinder monitoring
* Automated stock procurement
* Cloud deployment
* Mobile application
* Real-time notifications
* Advanced optimization algorithms
* Predictive maintenance for delivery vehicles

---

# Project Status

🚧 **Under Development**

The project is being developed as an academic **Operating Systems + DBMS integrated project**.

---

# Team

**GasGrid — LPG Supply Network & Resource Allocation System**

Developed as a B.Tech CSE project with focus on:

* Operating Systems
* Database Management Systems
* Resource Allocation
* Process Scheduling
* Database Transactions
* Supply Network Management

---

## License

This project is developed for academic and educational purposes.
