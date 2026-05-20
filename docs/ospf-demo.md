# OSPF Demo: Four Neighbors

This demo shows how Mini Router OS simulates a simplified OSPF shortest-path selection among four neighbors.

## Scenario

We add four OSPF neighbors, each advertising network prefixes with different direct costs.

- `r1` advertises `10.0.1.0/24` at direct cost `5`
- `r2` advertises `10.0.1.0/24` at direct cost `2`
- `r3` advertises `10.0.2.0/24` at direct cost `5`
- `r4` advertises `10.0.1.0/24` and `10.0.3.0/24` at direct cost `1`

We also configure a multi-hop topology between neighbors so the route to `r3` can be learned more cheaply through `r2`.

- `r2` <-> `r3` cost `1`
- `r3` <-> `r4` cost `1`

## Commands

### Add neighbors

```bash
curl -X POST -H "Content-Type: application/json" \
  -d '{"id":"r1","cost":5,"advertised":["10.0.1.0/24"]}' \
  http://localhost:8000/api/ospf/neighbors

curl -X POST -H "Content-Type: application/json" \
  -d '{"id":"r2","cost":2,"advertised":["10.0.1.0/24"]}' \
  http://localhost:8000/api/ospf/neighbors

curl -X POST -H "Content-Type: application/json" \
  -d '{"id":"r3","cost":3,"advertised":["10.0.2.0/24"]}' \
  http://localhost:8000/api/ospf/neighbors

curl -X POST -H "Content-Type: application/json" \
  -d '{"id":"r4","cost":1,"advertised":["10.0.1.0/24","10.0.3.0/24"]}' \
  http://localhost:8000/api/ospf/neighbors
```

### Configure neighbor links

```bash
curl -X POST -H "Content-Type: application/json" \
  -d '{"node_a":"r2","node_b":"r3","cost":1}' \
  http://localhost:8000/api/ospf/links

curl -X POST -H "Content-Type: application/json" \
  -d '{"node_a":"r3","node_b":"r4","cost":1}' \
  http://localhost:8000/api/ospf/links
```

### Run OSPF

```bash
curl -X POST http://localhost:8000/api/ospf/run
```

### View the routing table

```bash
curl http://localhost:8000/api/routing-table
```

## Expected behavior

`Mini Router OS` will learn routes from all four neighbors and choose the best route using the following selection rules:

1. Longest-prefix match first
2. Lowest metric when multiple routes cover the same prefix

In this example, three neighbors advertise `10.0.1.0/24`. The router should pick the route from `r4` because it has the lowest direct cost.

Because the OSPF topology links connect `r2` to `r3`, the learned route to `10.0.2.0/24` can be chosen through a multi-hop path with lower total cost than the direct link to `r3`.

The routing table should include learned routes such as:

- `10.0.1.0/24` via `r4` (metric `1`)
- `10.0.2.0/24` via `r3` (metric `3`)
- `10.0.3.0/24` via `r4` (metric `1`)

## Notes

This demo uses a simplified OSPF model. It demonstrates neighbor-based route learning and cost-based route selection, but does not simulate full link-state database flooding or multi-hop path computation.

For a real OSPF implementation, the algorithm would also exchange link-state advertisements (LSAs) and compute a shortest-path tree across multiple router hops.
