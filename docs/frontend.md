# Frontend

The frontend is a small React-based single-page application served directly from the backend's static files.

## Key files

- `frontend/index.html` — application shell and CDN imports for React, ReactDOM, and Babel
- `frontend/app.js` — React app logic, API integration, and UI state
- `frontend/style.css` — page and component styling

## How it works

The page loads React and ReactDOM from CDN and renders the app into `<div id="root">`.

`frontend/app.js` performs the following actions:

- Calls the backend API to load status, interface list, routes, routing table, and OSPF state
- Displays sections for:
  - status summary
  - interface management
  - static route management
  - packet forwarding simulation
  - routing table view
  - OSPF neighbor management
  - learned routes via OSPF
- Uses native browser `fetch()` for API communication
- Uses React hooks (`useState`, `useEffect`) for state and lifecycle

## UI features

- Create and delete virtual interfaces
- Create and delete static routes
- Run packet forwarding tests
- View the active merged routing table (static + OSPF routes)
- Add/delete OSPF neighbors
- Configure OSPF neighbor links for multi-hop topology
- View a topology diagram of neighbors and links
- Trigger OSPF recomputation and view learned OSPF routes

## Notes

Because React is loaded via CDN, there is no frontend build step required to run the app. The browser can load the page directly from the backend static server.
