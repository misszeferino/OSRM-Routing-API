# OSRM - Open Source Routing Machine & Open Street Maps - Configuration

In a past project, the need arose to improve driving distance calculations, which were initially based on the crow-fly (straight-line) method. This approach lacked accuracy for real-world applications, as it did not account for road networks, traffic conditions, or vehicle constraints.

A key challenge was developing an alert system for electric vehicle drivers to notify them when their remaining battery charge might be insufficient to return to the starting point. To address this, I conducted research and testing on OSRM (Open Source Routing Machine) as a possible solution. The testing was carried out both on a local machine and on a Google Cloud instance to evaluate performance, scalability, and deployment feasibility.

OSRM, an open-source routing engine built on OpenStreetMap data, enables precise route calculations and travel time estimates using real road networks. This repository documents the setup, configuration, and usage of OSRM, including how to deploy an OSRM server with Docker and make routing requests via the OSRM API.

## OpenStreetMap
OpenStreetMap is an open-source alternative to Google Maps, where maps are maintained and updated by volunteers. The platform primarily focuses on maintaining the maps rather than extracting and analyzing data from them.

## OSRM (Open Source Routing Machine)
OSRM is an open-source software solution built on top of OpenStreetMap’s geographical data, capable of calculating routes between locations. It serves as a free alternative to the Google Maps API. You set up a server that accepts HTTP requests containing location information for stops, and it returns data depending on the type of response you’re looking for.

Several services are available, including:
- Distance matrix
- Fastest route
- Nearest road

OSRM uses two algorithms to calculate routes: Multi-Level Dijkstra (MLD) and Contraction Hierarchies (CH).

## Data Extraction Process
The extraction process takes raw data from the map file and formats it so that OSRM can easily access it to calculate routes. To download OpenStreetMap extracts, for example, from Geofabrik:

```bash
wget http://download.geofabrik.de/europe/germany/berlin-latest.osm.pbf


## Preprocessing the Extract
Preprocess the extract using the car profile and start a routing engine HTTP server on port 5000:

```bash
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-extract -p /opt/car.lua /data/berlin-latest.osm.pbf
```

The flag `-v "${PWD}:/data"` creates the directory `/data` inside the Docker container and makes the current working directory (`${PWD}`) available there. The file `/data/berlin-latest.osm.pbf` inside the container refers to `${PWD}/berlin-latest.osm.pbf` on the host.

## Preprocessing Pipelines
The easiest and quickest way to set up your own routing engine is by using Docker images. There are two preprocessing pipelines available:
- **Contraction Hierarchies (CH)**: Best for large distance matrices.
- **Multi-Level Dijkstra (MLD)**: Default option.

By default, MLD is recommended except for special use cases, such as very large distance matrices, where CH is still a better fit. Below is an explanation of the MLD pipeline. If you want to use the CH pipeline instead, replace `osrm-partition` and `osrm-customize` with a single `osrm-contract` and change the algorithm option for `osrm-routed` to `--algorithm ch`.

```bash
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-partition /data/berlin-latest.osrm
```

```bash
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-customize /data/berlin-latest.osrm
```

Note that `berlin-latest.osrm` has a different file extension.

## Initializing an API Endpoint
The following command initializes an API endpoint to which you can send coordinates to obtain distance measurements and travel times. By default, it is mapped to port 5000.

```bash
docker run -t -i -p 5000:5000 -v "${PWD}:/data" osrm/osrm-backend osrm-routed --algorithm mld /data/berlin-latest.osrm
```

## Making Requests to the HTTP Server
Example request:

```bash
curl "http://127.0.0.1:5000/route/v1/driving/13.388860,52.517037;13.385983,52.496891?steps=true"
```

```
