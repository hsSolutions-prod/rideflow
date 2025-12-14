# RideFlow — High-Level Solution Architecture

This document describes the high-level system architecture of RideFlow, a
ride-sharing mobility platform designed around event-driven coordination,
stateless services, and real-time location processing.

The architecture emphasizes scalability, fault tolerance, and clear service
boundaries over feature completeness.

---

## Architecture Overview

RideFlow is composed of client applications, backend services, an asynchronous
event backbone, and supporting infrastructure layers.

Key principles:
- Single entry point via API Gateway
- Separation of Rider and Driver (Partner) workflows
- Event-driven communication using Kafka
- Derived state for fast matching using Redis
- Externalized routing and map visualization

---

## High-Level Architecture Diagram

```mermaid
flowchart LR

    %% =====================
    %% Client Applications
    %% =====================
    subgraph Clients
        RiderApp[User App]
        DriverApp[Partner App]
    end

    %% =====================
    %% Edge Layer
    %% =====================
    subgraph Edge
        Gateway[API Gateway]
    end

    %% =====================
    %% Backend Services
    %% =====================
    subgraph Services
        UserBackend[User Backend]
        PartnerBackend[Partner Backend]
        MatchingService[Matching Service]
    end

    %% =====================
    %% Event Backbone
    %% =====================
    subgraph Eventing
        Kafka[Kafka Event Bus]
        StateStore[Kafka State Store]
    end

    %% =====================
    %% Data Layer
    %% =====================
    subgraph Data
        Redis[Redis Cache]
        Postgres[(PostgreSQL)]
    end

    %% =====================
    %% Maps and Routing
    %% =====================
    subgraph Geo
        Routing[Routing Engine - GeoRush]
        Mapbox[Mapbox - Map Tiles]
    end

    %% =====================
    %% Observability
    %% =====================
    subgraph Observability
        Prometheus[Prometheus]
        Grafana[Grafana]
    end

    %% =====================
    %% Traffic Flow
    %% =====================
    RiderApp --> Gateway
    DriverApp --> Gateway

    Gateway --> UserBackend
    Gateway --> PartnerBackend

    %% =====================
    %% Event Flow
    %% =====================
    UserBackend --> Kafka
    PartnerBackend --> Kafka
    MatchingService --> Kafka

    Kafka --> StateStore
    StateStore --> Redis

    %% =====================
    %% Matching Logic
    %% =====================
    MatchingService --> Redis
    MatchingService --> Routing

    %% =====================
    %% Persistence
    %% =====================
    UserBackend --> Postgres
    PartnerBackend --> Postgres

    %% =====================
    %% Map Visualization
    %% =====================
    RiderApp -.-> Mapbox
    DriverApp -.-> Mapbox

    %% =====================
    %% Observability Wiring
    %% =====================
    Gateway --> Prometheus
    UserBackend --> Prometheus
    PartnerBackend --> Prometheus
    MatchingService --> Prometheus
    Prometheus --> Grafana
