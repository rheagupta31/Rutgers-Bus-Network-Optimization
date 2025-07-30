
🚍 **Rutgers Bus Network Optimization**

Developed a data-driven solution to optimize Rutgers University's campus bus system using real-world stop and route data. Applied network analysis and scheduling algorithms to improve travel efficiency, minimize wait times, and enhance service reliability for students and staff.

## Project summary

Our project summary can be found:

- as a notebook on `nbviewer`

https://nbviewer.org/gist/YOUR-GH-USERNAME/????????????????????????/

OR

- as a website:

https://moran-teaching.github.io/project-repo/????????????

## Accessing data

Our raw data was obtained directly from the **PassioGo Rutgers API**, which provides real-time information on campus bus locations, routes, and ETAs.

- **Data Source:** [`https://passiogo.com/mapGetData.php`](https://passiogo.com/mapGetData.php)
- **Format:** JSON
- **Update Frequency:** Live (every 10 seconds)

From the [`https://passiogo.com/mapGetData.php`](https://passiogo.com/mapGetData.php) API, we extracted the following fields for each active bus and stored them in `raw_bus_list`:

| Field       | Description                                          |
|-------------|------------------------------------------------------|
| `id`        | Unique identifier for the bus                        |
| `name`      | Display name or label of the bus                     |
| `route`     | Route name the bus is currently servicing            |
| `lat`       | Latitude coordinate of the bus                       |
| `lon`       | Longitude coordinate of the bus                      |
| `course`    | Direction of movement in degrees (0–360)             |
| `color`     | Route color used for frontend map visualization      |
| `paxLoad`   | Current passenger load (if available)                |
| `progress`  | Fractional progress along the route (computed value) |

> These fields were stored in a flat JSON structure and used as the basis for all downstream processing and enrichment steps.

## Python scripts / notebooks

The following scripts/notebooks were used produce the summary:

- `src/script.py`
- `notebooks/data_cleaning.ipynb`
- `notebooks/data_enrichment.ipynb`
- `notebooks/data_analysis.ipynb`

[Give a short description of what the notebooks contain, and their location in the git repo]

## Reproducibility

Provide a `requirements.txt` file with packages and versions of all python packages to run the analysis.

## Guide

### Summary
<br>

### Data Source and Nature

Our project relies on **real-time transit data** sourced from the official **PassioGo API**, the system Rutgers University uses to track its campus buses.

- **Source:** [`https://passiogo.com/mapGetData.php`](https://passiogo.com/mapGetData.php)  
  This API provides live information on buses, stops, and estimated arrival times.

- **Format:** Data is returned in **JSON** format with nested structures containing vehicle metadata, route details, and stop-level ETA predictions.

- **Key Data Attributes:**
  - **GPS coordinates** (latitude & longitude) of all active buses
  - **Route metadata** including route names, identifiers, and color codes
  - **Heading/course angle** indicating bus direction (0–360°)
  - **Passenger load** (`paxLoad`) when available
  - **Live ETAs** for buses arriving at each stop

- **Update Frequency:** The data updates approximately **every 10 seconds**, making it suitable for high-resolution, real-time analytics and visual tracking.

> This live stream of transit data served as the foundation for our bus tracking, bunching detection, and spacing optimization logic.

<br>

### Data Retrieval

Real-time data was retrieved using **HTTP GET/POST requests** to the PassioGo API at  
[`https://passiogo.com/mapGetData.php`](https://passiogo.com/mapGetData.php), handled via the custom backend script:

 [`backend/main.py`](./backend/main.py)

We utilized **three key API calls** to power our system:

1. **Bus Locations**  
   - **Endpoint:** `getBuses=1`  
   - **Purpose:** Retrieves live GPS positions, route assignments, and metadata for all active buses.

2. **Bus Stops**  
   - **Accessed via:** `passiogo.getSystemFromID(...).getStops()`  
   - **Purpose:** Returns stop metadata including name, ID, coordinates, and associated routes.

3. **Estimated Time of Arrival (ETA)**  
   - **Endpoint:** `eta=3`  
   - **Purpose:** Provides real-time arrival estimates of buses at specified stop IDs.

>  All API requests use randomly generated `deviceId` and `userId` values to avoid server-side caching and ensure fresh data retrieval.


<br>

###  Data Enrichment

Enrichment logic was implemented directly in the backend using:

 [`backend/route_geojson/`](./backend/route_geojson)
 [`backend/route_raw_polylines/`](./backend/route_raw_polylines)

Key enrichment steps included:

- **Computing Fractional Progress:**  
  Each bus’s GPS coordinates were projected onto its corresponding route polyline to calculate **fractional progress** (a value between 0 and 1), using custom utilities in `utils/progress.py`.

- **Bunching Detection:**  
  A bus was flagged as **bunched** (`isBunched = True`) if its progress gap with the next bus on the same route was below a dynamic threshold based on the number of active buses.

- **Wait Time Suggestions:**  
  When bunching was detected, the system recommended **wait times** (e.g., 30s to 120s) at the **next safe stop ahead**, based on the severity of the bunching gap.

- **Route Metadata Mapping:**  
  Each bus and stop was mapped to route-specific metadata like `safe_stop_ids` and `route_safe_stops_ordered`, enabling context-aware recommendations.

- **Manual Route Polyline Construction:**  
  For each route, we **manually tracked a bus for approximately one hour** and recorded its path using [GeoJSON.io](https://geojson.io).  
  These points were stored as GeoJSON files in `route_geojson/` and as raw coordinate arrays in `route_raw_polylines/`.

> This enriched structure was critical to enabling real-time progress computation, detecting bus clustering, and generating safe, actionable spacing interventions.


<br>

###  Summary Statistics

| Metric                    | Value                        |
|---------------------------|------------------------------|
| Avg. # of Active Buses    | 42                           |
| Avg. Buses per Route      | 4–6                          |
| % Routes with Bunching    | ~35% during peak hours       |
| Avg. Recommended Wait     | 45 seconds                   |
| Max Wait Recommendation   | 120 seconds                  |


<br>

### Key Visualizations

All visualizations and interactive features were implemented using **React Leaflet** in:

`bus-tracker-react/index.html`

---

- **Live Bus Progress Tracker**  
  Displays each bus’s progress `[0–1]` along the route polyline loop.  
  Insight: Clustering and uneven spacing can be spotted visually in real time.

- **Bunching Over Time (Time Series)**  
  Monitors the number of bunched buses at every 10-second update.  
  Insight: Useful for measuring how effective spacing logic is over time.

- **Interactive Route Filter (Leaflet UI)**  
  Allows users to isolate specific routes, highlighting only their buses, stops, and spacing recommendations.  
  Insight: Enables quick inspection and debugging of route-specific behaviors.

---

All visualizations were rendered dynamically using **React Leaflet**, with data fed from the FastAPI backend via API endpoints.



<br>

### Conclusion

Our project successfully delivered a real-time bus tracking system tailored to the Rutgers University campus, with an emphasis on solving the common issue of bus bunching. By integrating live transit data from the PassioGo API, enriching it with route-aware logic, and visualizing insights through an interactive React Leaflet interface, we created a robust tool that not only monitors but actively suggests corrective actions to restore optimal bus spacing.

The modular design of our backend and frontend allows for easy expansion, whether it’s dynamic re-routing, load-based decision-making, or integrating historical performance analytics. Most importantly, our system ensures better reliability for students and staff by minimizing unpredictable wait times caused by inefficient bus clustering.

This project demonstrates the potential of combining geospatial computation, real-time data streaming, and user-focused visualization to address logistical challenges in public transportation systems.

