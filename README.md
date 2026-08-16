# Decentralized Trust-Aware UAV Swarm Communication

A simulation-based research project for designing and evaluating a decentralized, trust-aware routing system for UAV (Unmanned Aerial Vehicle) swarms operating in highly dynamic wireless networks.

The project focuses on three problems in UAV ad-hoc networks: the topology keeps shifting because the UAVs never stop moving, links appear and disappear fast enough to break routing reliability, and one or more UAVs may behave maliciously.

Results come out of a network simulator, but the final presentation is a 3D UAV swarm simulation rather than packet traces or node graphs.

---

## 1. Project Objective

The objective is to develop a routing framework in which UAVs make routing decisions using local information instead of relying on one central controller.

Each UAV observes its neighboring UAVs and maintains information such as:

- UAV ID
- Position `(x, y, z)`
- Velocity and movement direction
- Neighbor UAVs
- Distance to neighbors
- Link quality / link stability
- Packet forwarding behavior
- Packet delivery statistics
- Packet drop statistics
- Trust score
- Routing-related state

The routing decision is therefore **distributed**. A UAV selects a next-hop neighbor according to network conditions, link quality, and trust instead of simply selecting the shortest path.

---

# 2. Problem Statement

Traditional routing approaches can perform poorly in UAV swarm networks because UAVs move continuously and the wireless topology changes rapidly.

A route that is valid at one time may become invalid a few seconds later.

The problem becomes more serious when a malicious UAV participates in routing. A malicious node may:

- Drop packets.
- Selectively forward packets.
- Advertise false routing information.
- Forward control traffic but drop data traffic.
- Perform blackhole-like behavior.
- Behave normally for some time and then become malicious.

The project aims to answer:

> **How can a decentralized UAV swarm maintain reliable communication and avoid untrusted forwarding nodes under dynamic mobility and malicious behavior?**

---

# 3. Core Theory

## 3.1 UAV Swarm Network

A UAV swarm is a collection of autonomous aerial nodes that communicate through wireless links.

The network is typically characterized by:

- High mobility
- Frequently changing neighbors
- Multi-hop communication
- Limited transmission range
- Variable link quality
- Distributed decision making

There is no assumption that every UAV has a fixed infrastructure connection.

---

## 3.2 MANET / FANET Concept

The project draws mainly on MANET (Mobile Ad Hoc Network) concepts, and more specifically on FANET (Flying Ad Hoc Network), which extends ad-hoc networking to highly mobile UAVs.

Compared to most terrestrial MANET scenarios, UAV movement is faster, fully three-dimensional, and changes the topology more often. Route stability has to be designed for from the start rather than treated as an afterthought.

---

## 3.3 Decentralized Routing

The project follows a decentralized architecture.

Instead of a central controller working out the complete route for the swarm, each UAV independently:

1. Discovers neighbors.
2. Measures link conditions.
3. Evaluates forwarding behavior.
4. Calculates trust.
5. Scores candidate next-hop UAVs.
6. Selects a suitable next hop.

This reduces dependence on a single controller and makes the system more suitable for distributed swarm operation.

---

# 4. Main System Architecture

```text
                    +----------------------+
                    |      UAV Swarm       |
                    +----------+-----------+
                               |
             +-----------------+-----------------+
             |                 |                 |
             v                 v                 v
      +-------------+   +-------------+   +-------------+
      | UAV Node 1  |   | UAV Node 2  |   | UAV Node N  |
      +-------------+   +-------------+   +-------------+
             |                 |                 |
             +-----------------+-----------------+
                               |
                               v
                    Neighbor Discovery
                               |
                               v
                     Mobility / Position
                               |
                               v
                       Link Evaluation
                               |
                               v
                      Trust Calculation
                               |
                               v
                     Route / Next-Hop Score
                               |
                               v
                        Packet Forwarding
                               |
                               v
                      Performance Metrics
```

---

# 5. What Each UAV Maintains

A logical UAV state can contain:

```text
UAV_ID
Position
Velocity
Neighbor_List
Distance_to_Neighbor
Link_Quality
Link_Stability
Packets_Sent
Packets_Received
Packets_Forwarded
Packets_Dropped
Packet_Delivery_Ratio
Trust_Score
Route_Cost
Current_Next_Hop
```

A simplified trust table can be represented as:

```text
Neighbor ID | PDR | Drop Rate | Link Quality | Trust | Status
------------|-----|-----------|--------------|-------|--------
UAV_02      | ... | ...       | ...          | ...   | Trusted
UAV_03      | ... | ...       | ...          | ...   | Suspicious
UAV_04      | ... | ...       | ...          | ...   | Trusted
```

---

# 6. Routing Theory

The project does not depend on one routing metric.

Instead, a candidate next-hop UAV can be evaluated using a **multi-factor routing score**.

A general form is:

```text
Score(i,j) =
    w1 * Trust(j)
  + w2 * LinkQuality(i,j)
  + w3 * LinkStability(i,j)
  + w4 * MobilitySuitability(i,j)
  - w5 * Distance(i,j)
  - w6 * Congestion(j)
```

Where:

- `Trust(j)` = trustworthiness of neighbor `j`
- `LinkQuality(i,j)` = estimated wireless link quality
- `LinkStability(i,j)` = probability that the link remains usable
- `MobilitySuitability(i,j)` = compatibility of movement between UAVs
- `Distance(i,j)` = physical separation
- `Congestion(j)` = forwarding load of the next hop
- `w1 ... w6` = configurable weights

The exact formula can be tuned during experiments.

---

# 7. Trust Management Theory

The central security component is a behavior-based trust model.

A UAV should not automatically trust a neighbor. Trust should be updated using observed forwarding behavior.

A basic trust model can be written as:

```text
Trust = alpha * DirectTrust
      + beta  * HistoricalTrust
      + gamma * RecommendationTrust
```

For the initial implementation, the project can focus mainly on **direct observation**, because it is simpler to validate experimentally.

A direct-trust estimate can use successful forwarding:

```text
ForwardingRatio =
    PacketsSuccessfullyForwarded /
    PacketsExpectedToBeForwarded
```

Then trust can be updated over time:

```text
Trust_new =
    lambda * Trust_old
  + (1 - lambda) * CurrentBehaviorScore
```

This introduces temporal smoothing and reduces abrupt trust fluctuations caused by isolated packet losses.

---

# 8. Malicious Attack Model

The project will explicitly model malicious UAV behavior.

## 8.1 Blackhole Attack

A malicious UAV advertises itself as a good forwarding node but does not forward received data packets.

Expected effect:

```text
Source -> Malicious UAV -> Destination
                  X
            Packet Dropped
```

The trust system should identify the difference between:

- packets received for forwarding
- packets actually forwarded

and gradually reduce the malicious node's trust.

---

## 8.2 Selective Forwarding

The UAV forwards some packets and drops others.

Example:

```text
Packet 1 -> Forward
Packet 2 -> Drop
Packet 3 -> Forward
Packet 4 -> Drop
```

This is more difficult than a simple blackhole because the attacker may appear partially reliable.

---

## 8.3 Optional Future Attacks

The repository can later support:

- Grayhole behavior
- False route advertisement
- Route manipulation
- Colluding malicious nodes
- On-off attacks

These should be added only after the baseline system is stable.

---

# 9. Mobility Model

UAV movement is a major part of FANET simulation.

Possible mobility models include:

### Random Waypoint

Useful as a basic benchmark but not always realistic for coordinated drone swarms.

### Gauss-Markov

Useful when correlated movement and smoother trajectories are required.

### 3D waypoint / trace-based mobility

Recommended for the final demonstration because UAVs can follow realistic 3D paths.

Example:

```text
UAV 1:  (0,0,100) -> (100,50,110) -> (200,100,120)
UAV 2:  (30,10,95) -> (120,70,105) -> (210,120,118)
UAV 3:  ...
```

For the visual final demo, a trace-based or waypoint-driven 3D movement model is preferable.

---

# 10. Network Simulation Stack

The core network simulation will use:

## NS-3

**ns-3** is the primary discrete-event network simulator.

It will be used to model:

- UAV nodes
- Wireless communication
- Routing
- Packet transmission
- Packet reception
- Packet loss
- Mobility
- Network attacks
- Performance metrics

The simulation layer provides reproducible experiments and quantitative results.

---

# 11. UAV Mobility and 3D Visualization

The project should separate:

```text
NETWORK SIMULATION
        |
        v
Simulation Trace / Metrics
        |
        v
3D UAV Visualization
```

The visualizer should not replace ns-3.

Instead:

**ns-3 = network truth**

**3D simulator = visual presentation**

The final visualization can show:

- UAV positions
- UAV movement
- Communication links
- Data packet flow
- Current source and destination
- Selected next-hop
- Malicious UAVs
- Trust state
- Route changes

A practical visualization stack can use:

- **Python**
- **Panda3D / PyOpenGL / Matplotlib 3D** for an early prototype

or a separate real-time engine such as:

- **Unity**
- **Godot**



---

# 12. Algorithms and Techniques

## 12.1 Neighbor Discovery

Each UAV periodically determines which UAVs are within communication range.

Basic calculation:

```text
distance =
sqrt((x2-x1)^2 + (y2-y1)^2 + (z2-z1)^2)
```

A UAV is considered a candidate neighbor if:

```text
distance <= communication_range
```

Additional link metrics can then be calculated.

---

## 12.2 Link Quality Estimation

Candidate metrics:

- Packet Delivery Ratio (PDR)
- Packet loss rate
- Received signal indicators when available
- Link duration
- Distance
- Relative velocity

A simple PDR:

```text
PDR = ReceivedPackets / SentPackets
```

---

## 12.3 Link Stability Estimation

A link between UAVs moving away from each other may become unusable quickly.

Route selection should factor in expected link duration, not just current connectivity.

The implementation can start with a simplified stability score based on:

- Distance
- Relative velocity
- Communication range

Then it can be improved using predicted link expiration time.

---

## 12.4 Trust Score Update

Trust is updated periodically based on forwarding behavior.

Possible components:

```text
Forwarding Success
Packet Drop Rate
Unexpected Route Behavior
Historical Reliability
```

A normalized trust value can be maintained in:

```text
0.0 <= Trust <= 1.0
```

Example:

```text
Trust >= 0.80   -> Trusted
0.50-0.80       -> Suspicious
Trust < 0.50    -> Untrusted
```

Thresholds should be treated as experimental parameters, not fixed universal values.

---

## 12.5 Next-Hop Selection

For each candidate neighbor:

1. Check whether the node is reachable.
2. Estimate link quality.
3. Estimate link stability.
4. Retrieve trust score.
5. Estimate forwarding load.
6. Calculate route score.
7. Select the highest-scoring candidate.

This becomes the main decentralized routing mechanism.

---

# 13. Algorithms Used

The initial repository will use the following algorithms / methods:

### Core algorithms

- Neighbor discovery
- Euclidean 3D distance calculation
- Packet Delivery Ratio calculation
- Link quality estimation
- Link stability estimation
- Exponential / weighted trust update
- Multi-criteria next-hop scoring
- Route selection
- Packet forwarding

### Routing baseline algorithms

The proposed method should be compared against at least one established routing approach.

Recommended baselines:

- AODV
- OLSR

The final selection depends on which routing implementations are easiest to reproduce in the chosen ns-3 version and project scope.

### Security algorithms

- Behavior-based trust calculation
- Malicious-node detection using forwarding statistics
- Trust threshold classification

### Optional advanced techniques

- Fuzzy logic for trust estimation
- Machine learning for malicious-node classification
- Reinforcement learning for adaptive routing

These are considered future extensions rather than mandatory first-stage components.

---

# 14. Libraries and Software

## Primary

| Tool / Library | Purpose |
|---|---|
| ns-3 | Network simulation |
| C++ | ns-3 simulation and routing implementation |
| Python | Analysis, visualization and experiment automation |
| NumPy | Numerical calculations |
| Pandas | Result processing |
| Matplotlib | Metrics plots |
| SciPy | Statistical analysis / mathematical utilities |
| Git | Version control |

## Visualization options

| Tool | Purpose |
|---|---|
| Python 3D | Initial UAV visualization |
| Panda3D / PyOpenGL | Interactive 3D visualization |
| Unity | Advanced final visualization |
| Godot | Open-source 3D visualization |


---

# 15. Experimental Metrics

The project must be evaluated using measurable network metrics.

## Packet Delivery Ratio

```text
PDR =
Packets successfully received at destination /
Packets generated by source
```

Higher is better.

---

## End-to-End Delay

Average time required for a packet to travel from source to destination.

Lower is better.

---

## Packet Loss Ratio

```text
PLR = 1 - PDR
```

Lower is better.

---

## Throughput

Amount of successfully delivered data per unit time.

```text
Throughput = DeliveredBits / SimulationTime
```

Higher is generally better.

---

## Routing Overhead

Measures the amount of routing/control traffic required to maintain communication.

Lower overhead is generally desirable.

---

## Trust Detection Performance

For malicious-node detection, measure:

- True Positive Rate
- False Positive Rate
- Detection Accuracy
- Detection Delay

These metrics help determine whether the trust system actually improves security.

---

# 16. Experimental Scenarios

The repository should contain controlled experiments.

### Scenario A — Normal Network

```text
All UAVs behave normally.
```

Purpose:

- Validate basic connectivity
- Validate routing
- Establish baseline performance

### Scenario B — Blackhole Attack

```text
One or more UAVs drop forwarded packets.
```

Purpose:

- Measure attack impact
- Evaluate trust-based detection

### Scenario C — Selective Forwarding

```text
Malicious UAV drops only a percentage of packets.
```

Purpose:

- Evaluate detection against stealthier behavior

### Scenario D — High Mobility

Increase UAV velocity and observe:

- Route break frequency
- Delay
- PDR
- Trust stability

### Scenario E — Different Swarm Sizes

Example:

```text
10 UAVs
20 UAVs
30 UAVs
50 UAVs
```

Observe scalability.

---

# 17. Baseline vs Proposed System

The proposed system should be compared against a routing baseline.

Conceptually:

```text
                     UAV FANET
                        |
             +----------+----------+
             |                     |
             v                     v
       Baseline Routing      Trust-Aware Routing
             |                     |
             v                     v
       Normal Metrics        Security + Routing
             |                     |
             +----------+----------+
                        |
                        v
                  Compare Results
```


# 18. Project Phases

## Phase 1 — Network and Security Foundation

The first phase establishes the complete simulation environment.

### Stage 1
Set up:

- ns-3
- C++
- Python analysis environment
- Git repository

### Stage 2
Create a basic UAV swarm:

- Multiple UAV nodes
- 3D positions
- Mobility
- Wireless communication

### Stage 3
Implement / configure baseline routing.

### Stage 4
Collect:

- PDR
- Delay
- Throughput
- Packet loss
- Routing overhead

### Stage 5
Implement malicious-node behavior.

### Stage 6
Implement trust calculation.

### Stage 7
Integrate trust into next-hop selection.

### Stage 8
Compare baseline and proposed routing.

**Phase 1 output:**

A working ns-3 simulation proving that the trust-aware decentralized routing mechanism works.

---

# 19. Phase 2 — Optimization and 3D Demonstration

The second phase focuses on advanced behavior, evaluation and presentation.

### Stage 1
Improve link stability estimation.

### Stage 2
Improve trust update logic.

### Stage 3
Tune multi-factor route scoring.

### Stage 4
Test multiple attack intensities.

### Stage 5
Run large-scale experiments.

### Stage 6
Automate experiment execution using Python.

### Stage 7
Create a 3D UAV visualization.

### Stage 8
Export simulation traces from ns-3 to the visualization layer.

### Stage 9
Display:

- UAV movement
- Communication links
- Routes
- Packet flow
- Malicious UAV
- Trust status

### Stage 10
Generate final result graphs and comparison tables.

**Phase 2 output:**

A reproducible research prototype plus a visual 3D UAV swarm demonstration.

---

# 20. Suggested Repository Structure

```text
uav-swarm-trust-routing/
│
├── README.md
├── LICENSE
├── .gitignore
│
├── ns3/
│   ├── scratch/
│   │   ├── uav-swarm.cc
│   │   ├── trust-model.cc
│   │   ├── trust-model.h
│   │   ├── routing-score.cc
│   │   └── routing-score.h
│   │
│   ├── scenarios/
│   │   ├── normal.cc
│   │   ├── blackhole.cc
│   │   └── selective-forwarding.cc
│   │
│   └── results/
│
├── python/
│   ├── analysis/
│   │   ├── metrics.py
│   │   ├── compare_results.py
│   │   └── statistics.py
│   │
│   ├── visualization/
│   │   ├── swarm_viewer.py
│   │   └── trace_loader.py
│   │
│   └── experiments/
│       └── run_experiments.py
│
├── configs/
│   ├── scenario_normal.yaml
│   ├── scenario_blackhole.yaml
│   └── scenario_selective.yaml
│
├── docs/
│   ├── architecture.md
│   ├── methodology.md
│   └── experiments.md
│
└── data/
    ├── raw/
    └── processed/
```

---

# 21. Development Workflow

The recommended workflow is:

```text
Define Scenario
      ↓
Create UAV Mobility
      ↓
Create Wireless Network
      ↓
Configure Routing
      ↓
Generate Traffic
      ↓
Collect Packet Events
      ↓
Calculate Link Metrics
      ↓
Calculate Trust
      ↓
Select Next Hop
      ↓
Simulate Malicious Behavior
      ↓
Collect Results
      ↓
Analyze with Python
      ↓
Visualize UAV Swarm
```

---

# 22. Development Strategy

Do not implement the entire research system at once.

Build incrementally:

```text
Level 1
2–5 UAVs + basic communication

Level 2
10 UAVs + mobility

Level 3
Routing + traffic

Level 4
Metrics collection

Level 5
Malicious UAV

Level 6
Trust mechanism

Level 7
Trust-aware routing

Level 8
Large-scale experiments

Level 9
3D visualization

Level 10
Final evaluation
```

This makes debugging significantly easier.

---

# 23. Expected Final Output

The final project needs both quantitative research results and a visual demonstration.

### Quantitative output

Graphs comparing:

```text
PDR
Delay
Throughput
Packet Loss
Routing Overhead
Detection Accuracy
False Positive Rate
```

under different:

- UAV counts
- mobility speeds
- attack percentages
- malicious-node counts

### Visual output

A 3D UAV swarm scene showing:

```text
UAV positions
        ↓
Movement
        ↓
Neighbor links
        ↓
Route selection
        ↓
Packet forwarding
        ↓
Malicious UAV
        ↓
Trust degradation
        ↓
Route avoidance
```

---

# 24. What Makes the Project Research-Oriented

The drone visualization is the presentation layer, not the contribution. What makes this a research project is dynamic FANET routing combined with decentralized decision making, behavior-based trust, malicious-node detection, multi-criteria next-hop selection, and quantitative simulation, all tied together and shown through the 3D swarm view.



---

# 25. Recommended Minimum Version (MVP)

The minimum successful implementation should contain:

- 10–20 UAVs
- 3D mobility
- Wireless communication
- Baseline routing
- Decentralized neighbor information
- Blackhole attack
- Trust score
- Trust-aware next-hop selection
- PDR / delay / throughput measurement
- Python analysis
- Basic 3D visualization

---

# 26. Future Extensions

Possible future research directions:

- Fuzzy trust systems
- Machine-learning-based attack detection
- Deep reinforcement learning routing
- Multi-attacker scenarios
- Colluding UAV detection
- Energy-aware routing
- QoS-aware routing
- Link prediction
- Digital-twin visualization
- Real UAV hardware validation

---

# 27. Keywords

```text
UAV
FANET
MANET
UAV Swarm
Decentralized Routing
Trust-Aware Routing
Secure Routing
Malicious Node Detection
Blackhole Attack
Selective Forwarding
Neighbor Discovery
Link Stability
Packet Delivery Ratio
ns-3
Wireless Ad Hoc Network
3D UAV Simulation
Network Simulation
```

---

# 28. Project Summary

This project builds a decentralized, trust-aware communication framework for UAV swarms.

Each UAV acts as its own network participant: it watches its neighbors, checks link conditions, tracks forwarding behavior, estimates trust, and picks a next hop without relying on a central routing controller.

The network is modeled in ns-3, with C++ handling the simulation and routing logic and Python covering experiment automation, analysis, and visualization.

The project models malicious UAV behavior directly, including blackhole and selective forwarding attacks. A behavior-based trust mechanism flags unreliable UAVs and feeds that into route selection.

Altogether it pulls together FANET theory, decentralized routing, trust management, malicious-attack detection, ns-3 simulation, Python analysis, and 3D UAV visualization into one system — meant to stand as a reproducible research prototype, with the visual demo as one part of that, not the whole point.

---

## Status

**Phase 1:** Network + Routing + Trust + Attack Simulation

**Phase 2:** Optimization + Evaluation + 3D UAV Visualization

**Primary simulator:** ns-3

**Primary implementation language:** C++

**Analysis / visualization:** Python
