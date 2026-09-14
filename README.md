# Location-Routing Optimization

## Problem Statement

This project addresses the Capacitated Location-Routing Problem (LRP), which jointly decides which depots to open and how vehicles should route from them to serve all customers, subject to vehicle capacity. The objective is to minimize total cost — routing distance plus depot fixed costs. The problem is formulated as a mixed-integer program and solved to proven optimality using Google OR-Tools' CP-SAT solver, validated against a published benchmark result, and extended with a sensitivity analysis on the number of open depots and vehicle capacity.

## Data Source

Instance data is sourced from S. Barreto's Location-Routing Problem benchmark instances (2004), a public academic benchmark combining facility-location and vehicle-routing decisions:

> Barreto, S. (2004). *Location-Routing Problem benchmark instances.* Available at sweet.ua.pt/sbarreto

This project uses the **Perl83-12x2** instance: 12 customers, 2 candidate depots, both with capacity 280 and fixed cost 100. Barreto's published results report a proven-optimal cost of **204** for this instance under vehicle capacity 140.

## Formulation

**Decision variables:**
- `x[i,j] = 1` if a vehicle travels directly from node `i` to node `j`
- `y[d] = 1` if depot `d` is opened

**Objective:**
Minimize total routing distance plus depot opening (fixed) costs.

**Constraints:**
1. Each customer has exactly one incoming and one outgoing arc.
2. The number of vehicles leaving a depot equals the number returning to it; arcs to or from a depot are only allowed if that depot is opened.
3. Vehicle capacity and subtour elimination are enforced jointly using a load-tracking variable per customer: cumulative route load must stay within vehicle capacity, and load must strictly increase along a route, which also rules out disconnected sub-loops that never pass through a depot.
4. Optional bounds on the number of depots opened, used to compare forced single-depot vs. two-depot configurations.

Distances are scaled to integers before solving, since CP-SAT requires integer coefficients, and divided back down when reporting final cost.

## Results

| Configuration | Cost | Depots Opened |
|---|---|---|
| 1 depot, capacity 140 | **203.98** | Depot 1 |
| 2 depots, capacity 140 | 295.76 | Both |
| 1 depot, capacity 160 | 202.56 | Depot 2 |
| 2 depots, capacity 160 | 295.76 | Both |

The single-depot, capacity-140 result (203.98) matches Barreto's published proven-optimal solution (204) to within floating-point rounding, confirming the formulation is correct.

**Depot count:** Forcing a second depot open raises total cost by 45%, since the added fixed cost isn't recovered by shorter routes for this customer layout.

**Vehicle capacity:** Raising capacity from 140 to 160 slightly lowers cost in the single-depot case (fewer, more efficient routes), and even changes which depot is preferred, since the best trade-off between the two depots shifts once routes are restructured. In the two-depot case, cost is unchanged, since demand is already split enough that no route was near the capacity limit — the binding constraint there is the fixed depot cost, not vehicle capacity.

## How to Run

1. Open `location_routing_optimization.ipynb` in Google Colab.
2. Install dependencies: `!pip install ortools`
3. Upload `Perl83Cli12x2` and `Perl83Dep12x2` (included in `/data`) to the Colab environment.
4. Run all cells in order.

## Notes

- This is a self-directed learning project built to apply integer programming and optimization techniques to a supply-chain problem, following the same approach used in a related nurse-scheduling optimization project.
- Possible extensions: testing larger Barreto instances, comparing exact-solver results against published metaheuristic solutions on harder instances, or extending the formulation to allow variable vehicle capacities.
