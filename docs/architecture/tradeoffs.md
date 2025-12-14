# RideFlow — Architectural Trade-Offs

This document captures the key architectural trade-offs made while designing
RideFlow. The goal is to explain *why* certain approaches were chosen over
alternatives, rather than claiming a single “correct” design.

---

## Event-Driven Coordination vs Synchronous Orchestration

**Choice:** Event-driven workflows using Kafka  
**Alternative:** Synchronous REST-based service chaining

**Why this choice**
- Ride-sharing workflows are naturally asynchronous and long-lived
- Synchronous chains increase latency and amplify failure propagation
- Event-driven communication allows services to evolve independently

**Trade-off**
- Eventual consistency instead of immediate consistency
- Increased operational complexity around event handling

---

## Avoiding Distributed Transactions

**Choice:** Eventual consistency with compensating logic  
**Alternative:** Distributed ACID transactions (2PC / XA)

**Why this choice**
- Distributed transactions do not scale well under high concurrency
- Tight coupling between services reduces system resilience
- Ride assignment can tolerate brief inconsistencies

**Trade-off**
- Requires careful handling of duplicate or out-of-order events
- System behavior must be reasoned in terms of state transitions

---

## Kafka as Coordination Backbone

**Choice:** Kafka for event propagation and state streaming  
**Alternative:** Point-to-point messaging or direct database polling

**Why this choice**
- High throughput and durability
- Enables replay and recovery of workflows
- Supports stream processing and state stores

**Trade-off**
- Operational overhead of managing Kafka clusters
- Requires schema discipline and versioning

---

## Driver State Management via State Stores + Redis

**Choice:** Kafka state store with Redis as derived cache  
**Alternative:** Querying the primary database for driver state

**Why this choice**
- Driver location and availability change frequently
- Redis provides low-latency access during matching
- Kafka state store acts as the source for derived state

**Trade-off**
- Redis is not the source of truth
- Cache invalidation and synchronization must be handled carefully

---

## Stateless Services vs Stateful Services

**Choice:** Stateless backend services  
**Alternative:** Stateful services with in-memory session data

**Why this choice**
- Enables horizontal scaling and easier failover
- Simplifies deployment and autoscaling
- Reduces risk during instance restarts

**Trade-off**
- Requires external state management
- Slightly higher latency due to external calls

---

## Location-Aware Matching Using Routing Engines

**Choice:** Routing-based distance and ETA calculation  
**Alternative:** Simple radius or straight-line distance matching

**Why this choice**
- Real-world road networks affect travel time significantly
- Improves match quality in dense urban environments
- Avoids unrealistic proximity assumptions

**Trade-off**
- Additional latency during matching
- Routing calls must be rate-limited and optimized

---

## Redis as Derived State, Not Source of Truth

**Choice:** Redis for fast access only  
**Alternative:** Treat Redis as a primary datastore

**Why this choice**
- Prevents data loss on cache eviction or restart
- Keeps system behavior predictable
- Aligns with eventual consistency model

**Trade-off**
- Requires fallback strategies if Redis is unavailable
- Adds complexity to state reconstruction

---

## Separating Rider and Driver Backends

**Choice:** Distinct User and Partner backend services  
**Alternative:** Single monolithic backend

**Why this choice**
- Rider and driver workflows evolve independently
- Different scaling and traffic patterns
- Clear ownership of responsibilities

**Trade-off**
- Increased number of services to manage
- Requires well-defined event contracts

---

## Externalizing Maps and Routing

**Choice:** Use open-source routing engines and Mapbox tiles  
**Alternative:** Building custom mapping and routing logic

**Why this choice**
- Mature tooling already exists for geo-routing
- Reduces time-to-learn and operational risk
- Allows focus on core system design

**Trade-off**
- Dependency on external systems
- Limited control over routing internals

---

## What This Document Represents

These trade-offs reflect a design optimized for **clarity, scalability, and
resilience**, rather than feature completeness or operational simplicity.

Different constraints (regulatory, financial, or organizational) may lead to
different decisions in a production environment.
