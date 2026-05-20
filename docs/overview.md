# Overview

Mini Router OS is a lightweight router simulator built with a Python backend and a browser-based frontend.

The goal of this project is to simulate key router functions for development and educational use, including:

- Virtual interface management
- Static routing with longest-prefix-match behavior
- OSPF-like route learning via neighbor advertisements
- Packet forwarding simulation
- A simple React-based front-end GUI
- Persistent configuration and route state using SQLite

The project is structured into two main layers:

- `backend/`: the FastAPI server, database models, routing simulation, and API endpoints
- `frontend/`: a small React-based single-page application served from the backend

This repository is designed for easy local use without a dedicated frontend build system. React is loaded via CDN so the UI can run directly from the `frontend/` assets.
