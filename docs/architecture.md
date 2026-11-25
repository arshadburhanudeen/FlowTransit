# System Architecture

## Purpose and scope

FlowTransit is a crowd-aware public transport decision-support platform for
bus passengers travelling along Colombo’s Galle Road corridor between
Colombo Fort and Mount Lavinia.

The MVP supports direct bus options between corridor stops. Bus locations
and passenger counts are simulated because no open live bus feed is
available to the project.

## System context

- **Passengers** search between two stops and compare direct bus options.
- **Data maintainers** maintain route, stop, stop-order, travel-time, and
  fare data in version-controlled project files.
- **The bus simulator** publishes simulated bus positions and passenger
  counts.
- **The team** builds, tests, packages, deploys, and operates the system.

## Architecture

FlowTransit uses a three-tier architecture with a modular monolith:

1. **Client tier:** an installable React web app built with Vite, designed
   for phone screens. Leaflet and OpenStreetMap provide the map.
2. **Application tier:** Nginx handles public HTTPS traffic, serves the web
   app, and forwards API requests to one Spring Boot application. The app is
   organized into modules and deployed as one unit.
3. **Data tier:** PostgreSQL stores routes, stops, route stop order, travel
   times, fare stages, bus types, and bus records. PostgreSQL is reachable
   only from the application tier.

The bus simulator runs in a separate container and publishes updates through
the Mosquitto MQTT broker. The Spring Boot tracking module consumes those
updates and maintains the latest bus state in memory.

## Application modules

The Spring Boot application contains these logical modules:

- **Transit data:** reads route, stop, stop-order, travel-time, and fare data.
- **Journey planner:** finds direct routes serving an origin and destination
  in the requested direction.
- **Fare estimation:** calculates fares from route fare stages.
- **Live tracking:** receives bus updates and serves the latest bus state.
- **Crowd estimation:** calculates occupancy from passenger count and bus
  capacity, then classifies it as Low, Medium, or High.
- **Option comparison:** ranks options as soonest, cheapest, least crowded,
  or balanced, and explains the trade-off between the top options.

Each application module uses controller, service, and repository layers.
Modules own their tables and expose Java interfaces to other modules.
Ranking modes use a strategy boundary so one mode can change independently
of the others.

## Key data flows

1. The passenger app sends origin and destination stops to the API.
2. The journey planner finds direct options and combines arrival estimate,
   travel time, fare, and crowd level.
3. The app polls the API for bus positions and crowd levels every five
   seconds while the map is open.
4. The simulator publishes position and passenger-count updates to Mosquitto
   every few seconds.
5. The tracking module consumes updates and recalculates crowd levels.
6. A scheduled task marks a bus stale after 60 seconds without an update.
   The scheduled check runs every 10 seconds.
7. Route, stop, and fare changes are loaded into PostgreSQL through Flyway
   migrations during deployment.

## Data and freshness

Route, stop, and fare information is stored as version-controlled data and
database migrations so the database can be rebuilt from the repository.
Adding a route should require data changes rather than application code
changes.

The first release keeps the latest simulated bus state in application memory;
it does not persist the high-volume update stream. Bus data older than
60 seconds must be marked stale and must not be presented as live.

## Security and network boundaries

Only Nginx is reachable from the public internet. The application and
database remain behind that entry point, and the database is on a private
network. Public traffic uses HTTPS. The server validates input. The MVP has
no passenger accounts and stores no personal passenger data. Secrets are
provided at deployment time and are not stored in the repository.

## Deployment and operations

The proposed deployment is one AWS EC2 instance running Docker Compose with
five containers: Nginx, the FlowTransit application, the bus simulator,
PostgreSQL, and Mosquitto.

GitHub Actions builds and tests the application, simulator, and frontend;
packages container images; publishes images to GitHub Container Registry;
deploys them to EC2; and verifies health endpoints after deployment. A
failure at any stage stops the release.

## Quality targets

- Journey search returns within two seconds on MVP data.
- A bus position reaches the map within ten seconds of being sent.
- Bus data older than 60 seconds is never shown as live.
- Every container has a health check and restarts automatically.
- The map is usable at phone width and conveys crowd levels with words as
  well as colour.
- The deployment stays within the project’s cloud-credit budget.

## Known trade-offs

The MVP uses one server, has no failover, and has no offline mode. If live
tracking is unavailable, the app reports that instead of presenting old
positions as current. Simulated results show system behavior, not real-world
passenger outcomes.

## Decisions to confirm

- Java build tool: Maven or Gradle.
- Bus simulator implementation language and MQTT message schema.
- Exact route, stop, fare, and travel-time source records.
- How PostgreSQL data is persisted in local and production deployments.