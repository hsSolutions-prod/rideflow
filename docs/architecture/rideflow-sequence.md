# RideFlow — Ride Request, Location Streaming & Driver Matching

This document describes the end-to-end sequence flow for how RideFlow processes
driver location updates and matches riders to nearby available drivers using an
event-driven architecture.

---

## Sequence Flow Overview

- Drivers continuously stream location and availability updates.
- Driver state is maintained in Kafka state stores and synchronized to Redis.
- Rider ride requests trigger an asynchronous matching workflow.
- Routing APIs are used to calculate proximity and ETA before driver selection.
- All coordination between services is handled via events to avoid distributed transactions.

---

## Ride Request & Matching Sequence

```mermaid
sequenceDiagram
    participant RiderApp as Rider App
    participant DriverApp as Driver App
    participant Gateway as API Gateway

    participant UserBackend as User Backend
    participant PartnerBackend as Partner Backend
    participant MatchingService as Matching Service

    participant EventBus as Kafka / Event Bus
    participant StateStore as Kafka State Store
    participant Redis as Redis Cache
    participant RoutingAPI as Routing Engine (OSRM)

    %% Login / Signup (simplified)
    RiderApp ->> Gateway: Login / Signup
    Gateway ->> UserBackend: Authenticate Rider

    DriverApp ->> Gateway: Login / Signup
    Gateway ->> PartnerBackend: Authenticate Driver

    %% Continuous Driver Location Streaming
    DriverApp -->> PartnerBackend: Upload location + availability
    PartnerBackend -->> EventBus: DriverLocationUpdated
    EventBus -->> StateStore: Update driver state
    StateStore -->> Redis: Sync active drivers

    %% Rider requests a ride
    RiderApp ->> Gateway: Request Ride
    Gateway ->> UserBackend: Create Ride (PENDING)

    UserBackend -->> EventBus: RideRequested
    EventBus -->> MatchingService: Consume RideRequested

    %% Matching logic
    MatchingService ->> Redis: Fetch nearby available drivers
    MatchingService ->> RoutingAPI: Calculate distance & ETA
    RoutingAPI -->> MatchingService: Route metrics

    MatchingService -->> EventBus: DriverMatched
    EventBus -->> UserBackend: Update Ride (ASSIGNED)
    EventBus -->> PartnerBackend: Notify selected driver

    %% Driver accepts ride
    DriverApp ->> Gateway: Accept Ride
    Gateway ->> PartnerBackend: Confirm Assignment
    PartnerBackend -->> EventBus: RideAccepted
    EventBus -->> UserBackend: Finalize assignment

