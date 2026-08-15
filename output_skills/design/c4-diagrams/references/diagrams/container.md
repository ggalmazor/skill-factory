# Container Diagram

Zooms into a single software system to show its high-level technical structure: the deployable/runnable units and how they communicate.

## Scope

A single software system.

## Elements

- **Primary**: Containers within the system (web apps, APIs, databases, message queues, file stores, etc.)
- **Supporting**: People and external software systems that interact with the containers

## Audience

Technical people — developers, architects, operations staff. Answers: "What are the major technical building blocks, and how do they talk to each other?"

## What "Container" Means

A container is a separately deployable or runnable thing:
- Server-side web application (Spring Boot, Express, Django)
- Client-side application (React SPA, iOS app, Android app)
- API service (REST API, GraphQL API, gRPC service)
- Database (PostgreSQL schema, MongoDB database, Redis)
- Message broker/queue (RabbitMQ, Kafka topic, SQS)
- File storage (S3 bucket, local filesystem)
- Serverless function (Lambda, Cloud Function)
- Background worker/job processor

NOT Docker containers (though a Docker container might host a C4 container).

## What to Include

- Every container with its name, technology, and responsibility
- Inter-container communication with protocol/technology ("JSON/HTTPS", "JDBC", "AMQP", "gRPC")
- External people and systems from the Context diagram, now connected to specific containers
- System boundary drawn around all containers to show what's in scope

## What NOT to Include

- Deployment infrastructure (load balancers, clustering, CDNs) — that's for Deployment diagrams
- Internal structure of containers (components, classes) — that's the next zoom level
- Replication or failover details

## Relationship Labels

At the container level, labels should describe **what flows** AND **how it flows**. Both the business intent and the protocol/technology matter.

Bad → Good:
- `"Uses"` → `"Reads order data from [JDBC]"`
- `"HTTPS"` → `"Submits new orders to [JSON/HTTPS]"`
- `"Calls"` → `"Validates payment via [REST/HTTPS]"`
- `"gRPC"` → `"Streams location updates to [gRPC]"`
- `"Reads/writes"` → `"Persists user sessions to [Redis protocol]"`

Format: `"<verb phrase describing what flows> [<protocol>]"`. The verb phrase belongs in the label; the protocol can be in brackets, parentheses, or on its own line — pick one and stay consistent.

## Common Mistakes

- Showing deployment nodes (servers, Kubernetes pods) — this is logical architecture, not physical
- Missing technology labels on containers ("Database" is not enough; say "PostgreSQL 15")
- Missing protocol labels on inter-container relationships
- Protocol-only labels — `"HTTPS"` alone tells the reader nothing about what's being communicated
- Making it too big — if you have 20+ containers, split into focused views by domain or user journey
- Confusing containers with components — if it's not separately deployable, it's probably a component

Source: https://c4model.com/diagrams/container
