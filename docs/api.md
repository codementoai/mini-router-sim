# API Reference

This project exposes a REST API for router simulation and management.

## Base URL

`http://localhost:8000`

## Static endpoints

- `GET /api/status`
  - Returns counts of interfaces, routes, and config items.

- `GET /api/interfaces`
  - Returns all configured virtual interfaces.

- `POST /api/interfaces`
  - Creates a new interface.
  - Request body:
    ```json
    {
      "name": "eth0",
      "address": "192.168.1.1",
      "netmask": "255.255.255.0"
    }
    ```

- `DELETE /api/interfaces/{name}`
  - Deletes an interface by name.

## Route management

- `GET /api/routes`
  - Returns all configured static routes.

- `POST /api/routes`
  - Adds a new static route.
  - Request body:
    ```json
    {
      "destination": "10.0.0.0/24",
      "gateway": "192.168.1.254",
      "interface": "eth0",
      "metric": 1
    }
    ```

- `DELETE /api/routes/{route_id}`
  - Deletes a route by its numeric ID.

## Config management

- `GET /api/config`
  - Returns all stored config entries.

- `POST /api/config`
  - Stores or updates config values.
  - Request body:
    ```json
    {
      "key": "hostname",
      "value": "mini-router"
    }
    ```

## Packet forwarding

- `POST /api/forward`
  - Simulates packet forwarding for a destination IP.
  - Request body:
    ```json
    {
      "source": "192.168.1.10",
      "destination": "10.0.0.5"
    }
    ```
  - Returns the chosen route or a drop reason.

## Routing table

- `GET /api/routing-table`
  - Returns the active merged routing table containing static and learned OSPF routes.

## OSPF simulation

- `GET /api/ospf/status`
  - Returns the current OSPF state:
    - `enabled`
    - `neighbors`
    - `learned_routes`

- `GET /api/ospf/neighbors`
  - Returns configured OSPF neighbor entries.

- `POST /api/ospf/neighbors`
  - Adds an OSPF neighbor.
  - Request body:
    ```json
    {
      "id": "r2",
      "cost": 2,
      "advertised": ["10.10.0.0/16"]
    }
    ```

- `DELETE /api/ospf/neighbors/{id}`
  - Removes a neighbor.

- `GET /api/ospf/links`
  - Returns configured OSPF neighbor links for multi-hop path computation.

- `POST /api/ospf/links`
  - Adds a bidirectional OSPF link between two neighbors.
  - The router uses these links to compute multi-hop shortest paths to advertised networks.
  - Request body:
    ```json
    {
      "node_a": "r2",
      "node_b": "r3",
      "cost": 1
    }
    ```

- `DELETE /api/ospf/links/{node_a}/{node_b}`
  - Removes a bidirectional link.

- `POST /api/ospf/run`
  - Triggers OSPF recomputation and learns advertised routes.

## Example workflow

1. Create interface
2. Add static route
3. Add OSPF neighbor with advertised networks
4. Run OSPF
5. Fetch `/api/routing-table`
## OSPF 4-neighbor example

This example demonstrates a simplified shortest-path OSPF selection among four neighbors advertising overlapping networks.

### Step 1: add neighbors

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

### Step 2: run OSPF

```bash
curl -X POST http://localhost:8000/api/ospf/run
```

### Step 3: inspect the routing table

```bash
curl http://localhost:8000/api/routing-table
```

Expected result (simplified):

```json
[
  {
    "id": null,
    "destination": "10.0.1.0/24",
    "gateway": "r4",
    "interface": "ospf-r4",
    "metric": 1,
    "source": "ospf"
  },
  {
    "id": null,
    "destination": "10.0.2.0/24",
    "gateway": "r3",
    "interface": "ospf-r3",
    "metric": 3,
    "source": "ospf"
  },
  {
    "id": null,
    "destination": "10.0.3.0/24",
    "gateway": "r4",
    "interface": "ospf-r4",
    "metric": 1,
    "source": "ospf"
  }
]
```

Because `r4` advertises `10.0.1.0/24` with the lowest cost, that route wins over the same network from `r1` and `r2`.
