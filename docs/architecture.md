# Architecture

## Product Choice

- **Product name:** Yandex Go
- **Link:** https://go.yandex/
- **Description:** Yandex Go is a ride-hailing and delivery platform that connects users with drivers and couriers for trips and goods delivery.

## Main components

![Yandex Go Component Diagram](diagrams/out/yandex-go/architecture-component/Component%20Diagram.svg)

<details><summary>Rendered image (click to open)</summary>

![Yandex Go Component Diagram](diagrams/out/yandex-go/architecture-component/Component%20Diagram.svg)

</details>

PlantUML: [Yandex Go Component Diagram Code](diagrams/src/yandex-go/architecture-component.puml)

Selected components:

1. **API Gateway** — Entry point for client requests; routes traffic to the right backend services.
2. **User Service** — Handles authentication, session validation, and user profile data.
3. **Dispatch Service** — Manages ride orders, driver matching, and order lifecycle (e.g. searching, assigned).
4. **Payment Service** — Processes payments, talks to the payment gateway, and records transaction status.
5. **Maps & Routing Service** — Provides route and traffic data, often by integrating with external maps.

## Data flow

![Yandex Go Sequence Diagram](diagrams/out/yandex-go/architecture-sequence/Sequence%20Diagram.svg)

PlantUML: [Yandex Go Sequence Diagram Code](diagrams/src/yandex-go/architecture-sequence.puml)

**Chosen group:** 3. Booking & Async Dispatch

**What happens:** The user confirms a ride from the app. The app calls the API Gateway to create an order. The Gateway calls the Dispatch Service, which checks the payment method with the User Service, saves the order (status SEARCHING) in the DB, and uses the cache for geospatial driver search. After matching a driver, it updates the order to ASSIGNED, writes to the DB, and publishes a RideAssigned event to Kafka. The Push Service consumes that event and sends a push to the app; the app shows the driver on the map.

**Components and data:** API Gateway and Dispatch Service (order creation, payment_id, class). Dispatch Service and User Service (payment method validation). Dispatch Service and Operational DB (order persistence and status updates). Dispatch Service and State Cache (geospatial query, candidate drivers). Dispatch Service and Kafka (RideAssigned event). Kafka and Push Service (event consumption). Push Service and Mobile App (push “Driver Found”).

## Deployment

![Yandex Go Deployment Diagram](diagrams/out/yandex-go/architecture-deployment/Deployment%20Diagram.svg)

PlantUML: [Yandex Go Deployment Diagram Code](diagrams/src/yandex-go/architecture-deployment.puml)

**Where components run:** Mobile app on the user’s phone; web app in the browser. API Gateway runs behind a load balancer in Yandex Cloud. Core services (User, Payment, Dispatch, Pricing, Maps & Routing, Notification) run in a Kubernetes cluster. Kafka runs in a message broker cluster; Redis in a cache cluster. YDB and ClickHouse run in a data storage cluster. Yandex Pay and Yandex Maps APIs run in a separate Kubernetes cluster marked as external.

## Assumptions

- The pricing service uses demand/supply or similar signals (e.g. from cache/analytics) to compute or adjust ride prices.
- The dispatch service uses the cache for geospatial driver lookup to quickly find nearby drivers.

## Open questions

- How is load balanced across microservices in production?
- How does the system behave under very high demand (e.g. New Year) in terms of scaling and limits?
