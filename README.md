# Location-Routing Optimization

This project started with a benchmark-replication question: **can I formulate a small capacitated location-routing problem and recover the published solution before changing any assumptions?**

I use Barreto's Perl83-12x2 instance, which has 12 customers and 2 candidate depots. The model makes depot-opening and routing decisions together rather than solving facility location and vehicle routing separately.

## What the model decides

The CP-SAT formulation chooses:

- which depots to open;
- which depot serves each customer; and
- the customer sequence on each vehicle route.

The constraints enforce customer service, vehicle capacity, depot capacity, route continuity, and consistency between a customer's assigned depot and the route that serves that customer. I added the assignment/depot-consistency constraints after checking the original formulation more closely; without them, a multi-depot formulation can permit route structures that are not valid for the LRP.

## Benchmark check

For the published setting with vehicle capacity `Q = 140` and one depot open, the model returns **203.98**, matching the reported benchmark value of **204** after the distance scaling/rounding used in the notebook.

| Setting | Objective | Depot choice |
|---|---:|---|
| 1 depot, Q = 140 | **203.98** | Depot 1 |
| 2 depots, Q = 140 | 295.76 | Both |
| 1 depot, Q = 160 | 202.56 | Depot 2 |
| 2 depots, Q = 160 | 295.76 | Both |

The sensitivity exercise is deliberately small. At `Q = 140`, forcing both depots open increases the objective by about **45%** relative to the best one-depot solution. Increasing capacity to 160 slightly changes the best one-depot solution and switches the preferred depot. The two-depot objective is unchanged at the two capacities tested.

## Files

- `location_routing_optimization.ipynb` — formulation, solution, route extraction, and sensitivity runs
- `data/Perl83Cli12x2` — customer data
- `data/Perl83Dep12x2` — depot data

Data source: S. Barreto, Location-Routing Problem benchmark instances (2004).

## Run it

```bash
pip install -r requirements.txt
jupyter notebook location_routing_optimization.ipynb
```

Run the notebook from top to bottom from the repository root so the relative `data/` paths resolve correctly.

## Scope

This is a benchmark study, not a general-purpose LRP solver. The next useful step would be to test larger instances and compare alternative formulations or heuristics once exact solution becomes more expensive.
