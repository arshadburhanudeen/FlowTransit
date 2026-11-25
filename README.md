# FlowTransit

FlowTransit helps bus passengers compare direct journey options by arrival
time, travel time, fare, and crowd level. It is designed for passengers
travelling along Colombo’s Galle Road corridor between Colombo Fort and
Mount Lavinia.

## MVP scope

The first release focuses on 3–5 bus routes that share stops along the
selected corridor. Bus positions and passenger counts are simulated because
the project has no access to an open live bus feed.

FlowTransit provides information to support a passenger’s choice. The
passenger decides which trade-off matters most.

## Core capabilities

- Search for direct bus options between two corridor stops.
- Compare estimated arrival time, travel time, fare, and crowd level.
- Rank options by soonest, cheapest, least crowded, or balanced.
- View routes, stops, and bus positions on an installable mobile web app.

## System overview

The proposed system uses a React and Vite installable web app, a Java and
Spring Boot API, PostgreSQL for route and fare data, and MQTT to deliver
simulated bus updates. Nginx provides the public HTTPS entry point.

See [System Architecture](docs/architecture.md) for component boundaries,
data flows, and operational requirements.

## Data and limitations

MVP bus positions and passenger counts are simulated. Results demonstrate
system behavior and do not measure real passenger behavior. The initial
release covers direct bus trips on the selected corridor; transfers, accounts,
payments, and the full Colombo network are outside the MVP.

## Development and operations
