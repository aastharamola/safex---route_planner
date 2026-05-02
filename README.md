# 🛡️ SafeX — Safe Route Planner

A safety-aware driving route planner that fetches multiple alternative routes between two locations and ranks them by a **safety score** calculated from real-world crime and accident data.

---

## 🚀 Features

- 🗺️ **Interactive map** powered by Leaflet.js + OpenStreetMap
- 🔀 **Up to 3 alternative routes** fetched via OpenRouteService API
- 🟢🟡🔴 **Color-coded safety ranking** — Green (safest), Yellow (moderate), Red (high risk)
- 📊 **Safety score (0–100)** calculated from nearby crimes and accidents
- 🔍 **Smart autocomplete** — searches local dataset index + Nominatim geocoder
- 📏 Shows **distance**, **estimated time**, and **safety score** for the best route
- 🚗 Animated loading overlay while route is being fetched

---

## 🏗️ Project Structure

```
safex-route-planner-upload/
├── frontend/
│   ├── login.html        # Entry/login page
│   ├── dashboard.html    # Main map UI
│   ├── app.js            # Route fetching, map drawing, autocomplete
│   ├── map.js            # Leaflet map initialization
│   ├── config.js         # API key config (frontend)
│   └── style.css         # All styles
├── backend/
│   ├── server.js         # Express API server
│   └── package.json      # Backend dependencies
├── cpp_engine/
│   ├── graph.h           # Graph data structure (Edge with distance + risk)
│   ├── router.h          # safestPath() function declaration
│   ├── router.cpp        # Dijkstra's algorithm implementation
│   ├── main.cpp          # C++ entry point
│   └── main.exe          # Compiled binary
├── data/                 # ⚠️ Not included — add CSV files here (see below)
├── .gitignore
└── README.md
```

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML, CSS, Vanilla JavaScript |
| Map | Leaflet.js + OpenStreetMap tiles |
| Routing API | OpenRouteService (ORS) |
| Geocoding | Nominatim (OpenStreetMap) |
| Backend | Node.js + Express.js |
| CSV Parsing | csv-parser |
| HTTP Client | Axios |
| C++ Engine | Dijkstra's algorithm (standalone) |

---

## ⚙️ How It Works

1. User enters a **Start** and **End** location
2. Backend calls the **OpenRouteService API** for up to 3 alternative routes
3. For each route, the server counts **crime and accident incidents** within a ~1 km bounding box
4. A **safety score** is computed using a logarithmic formula:
   ```
   safety = 100 - log10(1 + totalIncidents) × 25
   ```
5. Routes are ranked: highest safety first, shortest distance as tiebreaker
6. Results are drawn on the map with **color-coded polylines** and tooltips

---

## 📦 Setup & Installation

### Prerequisites
- [Node.js](https://nodejs.org/) (v18 or higher recommended)
- Internet connection (for ORS API + Nominatim)

### 1. Clone / Download the project

```bash
git clone <your-repo-url>
cd safex-route-planner-upload
```

### 2. Install backend dependencies

```bash
cd backend
npm install
```

### 3. Add datasets (Optional — for safety scoring)

Place the following CSV files in a `data/` folder at the project root:

```
safex-route-planner-upload/
└── data/
    ├── Crimes_-_2001_to_Present.csv     ← Chicago crime data
    └── US_Accidents_March23.csv         ← US traffic accidents
```

> ⚠️ These files are large and excluded from Git. Without them, the app still works but all safety scores will show **100** (no data to count).

**Download links:**
- [Chicago Crimes Dataset](https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-Present/ijzp-q8t2)
- [US Accidents Dataset](https://www.kaggle.com/datasets/sobhanmoosavi/us-accidents)

### 4. Start the server

```bash
cd backend
node server.js
```

### 5. Open in browser

```
http://localhost:3000
```

---

## 🗺️ Example Routes to Try

| Start | End |
|---|---|
| `Chicago O'Hare International Airport, Chicago, IL` | `Millennium Park, Chicago, IL` |
| `Times Square, New York, NY` | `Brooklyn Bridge, New York, NY` |
| `Los Angeles International Airport, CA` | `Hollywood Sign, Los Angeles, CA` |
| `Chicago, IL` | `Indianapolis, IN` |

> 💡 Tip: Type at least 2 characters to trigger autocomplete suggestions.

---

## 🔌 API Endpoints

### `GET /autocomplete?q=<query>`
Returns up to 10 location suggestions from the loaded crime/accident datasets.

**Response:**
```json
[
  { "name": "N Michigan Ave, Chicago, IL", "lat": 41.89, "lng": -87.62 }
]
```

---

### `POST /route`
Fetches and scores alternative driving routes.

**Request Body:**
```json
{
  "start": { "lat": 41.9742, "lng": -87.9073 },
  "end":   { "lat": 41.8827, "lng": -87.6233 }
}
```

**Response:** GeoJSON `FeatureCollection` with safety scores attached to each route feature:
```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "geometry": { ... },
      "properties": {
        "safety": 87,
        "safetyBreakdown": {
          "crimeCount": 120,
          "accidentCount": 34
        },
        "summary": { "distance": 28400, "duration": 1920 }
      }
    }
  ]
}
```

---

## 🧠 C++ Engine (Standalone)

The `cpp_engine/` folder contains a custom **Dijkstra's algorithm** implementation in C++ for graph-based pathfinding. Each edge has a `distance` and `risk` value, and the cost function is:

```
cost = distance + 2 × risk
```

> ℹ️ This module is currently standalone and not integrated with the Node.js backend. It can be extended in the future as a local offline routing engine.

To compile manually:
```bash
g++ -o main main.cpp router.cpp
./main
```

---

## 👤 Author

Built as a full-stack safety routing system combining real-world open datasets with a modern web interface.