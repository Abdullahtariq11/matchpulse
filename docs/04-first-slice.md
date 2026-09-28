# MatchPulse First Vertical Slice

## 1. Goal

The goal of the first vertical slice is to build the smallest working
version of MatchPulse that can receive a football match event and update
a player's match statistics.

MatchPulse is intended to be a learning project for understanding
microservices and event-driven architecture.

The system will therefore be built incrementally rather than introducing
all infrastructure at the beginning.

The initial version will use simple synchronous communication between
services. More advanced infrastructure, such as a message broker, will
be introduced when its purpose and benefits can be demonstrated through
the limitations of the simpler architecture.

---

## 2. First Vertical Slice

The first vertical slice will support one basic football event:

    PASS

A simulator will generate a PASS MatchEvent and send it to MatchPulse.

For example:

    {
      "eventId": "event-1",
      "matchId": "match-1",
      "playerId": "player-8",
      "type": "PASS",
      "outcome": "SUCCESS",
      "sequenceNumber": 1,
      "matchTime": "12:35"
    }

After processing this event, MatchPulse should produce player statistics
similar to:

    playerId: player-8
    matchId: match-1
    passesAttempted: 1
    passesCompleted: 1

If another event is received:

    type: PASS
    outcome: FAILED

the statistics should become:

    passesAttempted: 2
    passesCompleted: 1

This is intentionally a very small feature.

Its purpose is to prove the complete event-processing flow before
additional event types and infrastructure are introduced.

---

## 3. Initial Architecture

The first implementation will use two Spring Boot microservices:

1. Event Processing Service
2. Statistics Service

The initial flow is:

    Simulator
        |
        | HTTP
        v
    Event Processing Service
        |
        | HTTP
        v
    Statistics Service
        |
        v
    PostgreSQL

The services have separate business responsibilities and separate data
ownership.

---

## 4. Event Processing Service

The Event Processing Service is responsible for receiving football
events from an event source.

Initially, the event source will be the MatchPulse simulator.

Its responsibilities will include:

- receiving MatchEvents
- validating incoming events
- identifying event type and outcome
- forwarding valid events for statistics processing

As the project evolves, this service may also become responsible for:

- duplicate event detection
- event ordering
- missing event detection
- event recovery
- failed event handling

These capabilities will be introduced incrementally rather than
implemented immediately.

The first version will expose an HTTP endpoint conceptually similar to:

    POST /events

The exact API contract will be defined during implementation.

---

## 5. Statistics Service

The Statistics Service is responsible for maintaining calculated player
statistics for each match.

It receives MatchEvents from the Event Processing Service and applies
the appropriate statistical changes.

For the first vertical slice, only PASS events need to be supported.

For example:

    PASS + SUCCESS

produces:

    passesAttempted += 1
    passesCompleted += 1

While:

    PASS + FAILED

produces:

    passesAttempted += 1

The Statistics Service owns PlayerMatchStats.

Additional statistics and event types will be introduced after the
initial PASS flow works correctly.

---

## 6. Service Communication

The first version will use synchronous HTTP communication between the
Event Processing Service and Statistics Service.

The flow is:

    Simulator
        |
        | HTTP
        v
    Event Processing Service
        |
        | HTTP
        v
    Statistics Service

A message broker will intentionally not be introduced in the first
version.

This allows the initial system to demonstrate the behavior and
limitations of synchronous service-to-service communication.

After the basic event-processing pipeline works, MatchPulse will
evaluate introducing a message broker such as Kafka or RabbitMQ.

The future architecture may evolve toward:

    Simulator
        |
        v
    Event Processing Service
        |
        | Publish Event
        v
    Message Broker
        |
        | Consume Event
        v
    Statistics Service

The decision between Kafka, RabbitMQ, or another approach will be made
based on MatchPulse requirements rather than selecting a technology in
advance.

---

## 7. Persistence

PostgreSQL will be used from the beginning.

Persistence is required because MatchPulse needs events and calculated
statistics to survive application restarts and support historical
analysis.

The microservices will follow separate data ownership.

Conceptually:

    PostgreSQL Server
        |
        +-- event_db
        |
        +-- statistics_db

The Event Processing Service owns:

    event_db

The Statistics Service owns:

    statistics_db

Both databases may initially run within the same PostgreSQL server or
development container.

Physical infrastructure can therefore remain simple while logical data
ownership remains separated.

---

## 8. Data Ownership

Each microservice is responsible for its own data.

The Event Processing Service must not directly read from or modify the
Statistics Service database.

Similarly, the Statistics Service must not directly read from or modify
the Event Processing Service database.

Conceptually:

    Event Processing Service
              |
              v
          event_db


    Statistics Service
              |
              v
        statistics_db

Communication between services should occur through defined service
contracts rather than shared database tables.

This prevents the services from becoming tightly coupled through their
database schemas.

For example, changing the internal database structure of the Statistics
Service should not require changes to the Event Processing Service.

---

## 9. Why PostgreSQL

The MatchPulse domain contains strongly related data such as:

- matches
- players
- match events
- player match appearances
- player match statistics
- player match ratings

Many expected queries are naturally relational.

Examples include:

    Get all events for a match ordered by sequence number.

    Get all players who appeared in a match.

    Get a player's statistics for a particular match.

    Get a player's most recent Manchester United appearance.

    Get historical statistics for a player.

PostgreSQL therefore provides a natural starting point for the
MatchPulse data model.

A NoSQL database such as DynamoDB could support MatchPulse, but would
require earlier decisions around access patterns, partition keys, sort
keys, and denormalization.

Those concerns are not currently required to build the first vertical
slice.

---

## 10. Initial Success Criteria

The first vertical slice is complete when the following flow works:

    Simulator
        |
        | PASS event
        v
    Event Processing Service
        |
        | HTTP
        v
    Statistics Service
        |
        v
    PostgreSQL

Specifically:

1. The simulator can create a PASS MatchEvent.

2. The simulator can send the event to the Event Processing Service
   through HTTP.

3. The Event Processing Service can receive and validate the event.

4. The Event Processing Service can send the event to the Statistics
   Service.

5. The Statistics Service can process the PASS event.

6. A successful pass increments:

       passesAttempted
       passesCompleted

7. An unsuccessful pass increments:

       passesAttempted

8. The resulting PlayerMatchStats are persisted in PostgreSQL.

9. The persisted statistics can be retrieved and verified.

Once these conditions are satisfied, the first vertical slice is
considered complete.

---

## 11. What Is Intentionally Not Included Yet

The first vertical slice will not include:

- Kafka
- RabbitMQ
- Redis
- Elasticsearch
- WebSockets
- player rating calculations
- frontend live updates
- multiple event types
- advanced recovery mechanisms
- deployment architecture
- distributed tracing
- service discovery
- API gateway

These technologies and capabilities may be introduced later when a
specific MatchPulse requirement justifies them.

The goal is not to simulate a large production architecture before the
basic system exists.

---

## 12. Planned Incremental Evolution

MatchPulse will evolve through small working increments.

A possible progression is:

    Step 1
    PASS event over HTTP

        ↓

    Step 2
    Duplicate event protection

        ↓

    Step 3
    Additional football event types

        ↓

    Step 4
    Continuous match simulator

        ↓

    Step 5
    Event ordering and missing event detection

        ↓

    Step 6
    Event recovery and retry behavior

        ↓

    Step 7
    Evaluate synchronous HTTP limitations

        ↓

    Step 8
    Introduce a message broker if justified

        ↓

    Step 9
    Build frontend match/player views

        ↓

    Step 10
    Introduce live frontend updates

        ↓

    Step 11
    Historical analysis and previous-appearance comparison

        ↓

    Step 12
    Research and implement player ratings

This sequence is not a fixed project schedule.

The order may change as implementation reveals new requirements or
architectural problems.

---

## 13. Development Principle

MatchPulse will follow the development cycle:

    Understand
        ↓
    Decide
        ↓
    Document
        ↓
    Implement
        ↓
    Test
        ↓
    Review

Technologies should be introduced because they solve an understood
problem.

The project should not use infrastructure simply because that
technology is commonly associated with microservices.

The architecture is expected to evolve as the system grows and as the
limitations of earlier design decisions become visible.

The first vertical slice deliberately starts simple so that later
architectural changes can be understood rather than copied.