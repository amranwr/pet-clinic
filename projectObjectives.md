# Pet Clinic — System Design Practice Project

A personal learning project to practice system design, modern architecture, and the
latest Java Spring technologies. Goal: **skill-building, not shipping fast**.
The frontend is intentionally out of focus (see section 4).

> Guiding rule: **every feature should teach you something new.** Build in phases,
> start simple, then deliberately re-architect to feel the trade-offs yourself.

---

## 0. How We Work (Read This First — Context for AI Assistant)

**About me:** senior backend engineer (Java/Spring). Treat me as an experienced
engineer; do not explain basics or use unnecessary jargon.

**Working agreement:**
- **I do the work.** I create the artifacts (ERD, designs, code, ADRs) myself,
  then ask the assistant to review them.
- **The assistant reviews, it does not direct.** Do not tell me what to do or which
  step to take next. Do not produce the deliverable for me unless I explicitly ask.
- **Guide, don't prescribe.** Point me in the right direction, or show alternatives
  and trade-offs I can study and decide between. I make the decisions.
- **Reviews should be concrete:** flag real issues (correctness, consistency,
  risks), mention alternatives worth considering, and say what I got right.
- **Keep it short and plain.** No heavy theory or textbook vocabulary (e.g. DDD
  terminology) unless I ask or it is directly needed for a decision.
- **Pragmatic over academic:** prefer simple solutions; avoid over-engineering.
- **Task tracking:** tasks live in the Notion "pet clinic tasks" database.
  Add or update tasks only when I ask.

---

## 1. Domain Model (System Design Starts Here)

Model the domain before writing any code.

### Core entities
- **Owner** → has many **Pets**
- **Pet** → has **MedicalHistory**, **Vaccinations**
- **Vet** → has **Specialties**, a **Schedule**
- **Appointment** → links Owner/Pet/Vet + time slot + status
- **Visit** → outcome of an appointment (diagnosis, prescriptions)
- **Invoice / Billing** → tied to visits
- **Notification** → reminders

### Criteria
- Write the domain model + an **ERD** and a **context diagram** before coding.
- Practice **Domain-Driven Design**: identify bounded contexts —
  **Scheduling**, **Medical Records**, **Billing**, **Notifications**.

---

## 2. Architecture Criteria (Pick a Progression, Not a Destination)

Deliberately evolve the architecture so you *feel* why each step exists.

| Phase | Architecture | What you learn |
|-------|--------------|----------------|
| 1 | **Modular monolith** (Spring Boot, package-by-feature) | Clean layering, DDD boundaries, REST design |
| 2 | Add **async** (events, message queue) | Decoupling, eventual consistency |
| 3 | Extract 1–2 **microservices** (Notifications, Billing) | Service boundaries, inter-service comms, distributed data |
| 4 | Add **API gateway + service discovery** | Cross-cutting concerns, routing, resilience |

> **Criteria:** Start with the **modular monolith**. Split into microservices only when
> you can clearly articulate *why*. Most people jump to microservices too early and
> learn the wrong lessons.

---

## 3. Backend Criteria (Java + Spring)

- **API design**: RESTful resources, proper status codes, pagination, filtering,
  optional HATEOAS. Document with **OpenAPI / Swagger**.
- **Layering**: Controller → Service → Repository, with **DTOs** (never leak entities;
  use MapStruct mappers).
- **Persistence**: Spring Data JPA + **PostgreSQL**; schema via **Flyway/Liquibase**.
- **Validation**: Bean Validation (`@Valid`), custom validators.
- **Error handling**: global `@RestControllerAdvice`, consistent error payloads.
- **Security**: **Spring Security + JWT**, role-based (Owner, Vet, Admin).
  Later: **OAuth2 / OIDC** with Keycloak.
- **Testing**: unit (JUnit5 + Mockito), integration (**Testcontainers** for real Postgres),
  API tests. Aim for meaningful coverage, not 100%.
- **Async & events**: start with Spring `ApplicationEventPublisher`, then
  **Kafka / RabbitMQ** for appointment reminders.
- **Caching**: **Redis** for vet availability / frequently read data.
- **Scheduling**: `@Scheduled` or **Quartz** for reminder jobs.

---

## 4. Frontend Criteria (Delegated — Not a Learning Focus)

I'm a backend engineer; the frontend is **not** a learning goal here.

- **Approach**: generate the Angular frontend with **Spec Kit + an LLM** from the
  backend's OpenAPI spec and feature specs.
- **Stack**: Angular (latest), whatever Spec Kit scaffolds — no deep dive into
  signals, NgRx, or UI architecture.
- **Contract**: the backend's **OpenAPI** spec is the source of truth; the FE consumes it.
- **Scope**: minimal UI to exercise the backend (CRUD screens, login, appointment booking).
- **Time budget**: keep FE effort low; review generated code only as needed to verify
  the backend integration works.

---

## 5. Cross-Cutting "Latest Tech" Criteria

Pick a few and go deep rather than all shallow.

- **Containerization**: Dockerize backend, frontend, DB. `docker-compose` for local.
- **Orchestration**: deploy to **Kubernetes** (minikube/kind locally) — deployments,
  services, ingress, configmaps/secrets.
- **CI/CD**: GitHub Actions / GitLab CI — build, test, containerize, deploy.
- **Observability**: **Prometheus + Grafana** metrics, **structured logging**,
  distributed tracing (**OpenTelemetry + Jaeger**).
- **Resilience**: **Resilience4j** (circuit breakers, retries) once you have microservices.
- **Config**: externalized config, Spring Cloud Config or env-based, feature flags.

---

## 6. Process Criteria (Makes It Feel Real)

- **Git discipline**: feature branches, meaningful commits, PRs to yourself with
  self-review.
- **Documentation**: a `docs/` folder with **ADRs** (Architecture Decision Records).
- **Diagrams**: keep **C4 model** diagrams (context, container, component) updated.
- **Issue tracking**: use a board (e.g. Notion) to plan features as a backlog.
- **Definition of Done** per feature: coded + tested + documented + diagrammed.

---

## 7. Feature Roadmap (Incremental Complexity)

1. **MVP monolith**: owners, pets, vets CRUD + auth.
   *(layering, REST, JPA, security)*
2. **Appointment scheduling**: slot management, conflict detection.
   *(business logic, concurrency — vets double-booking is a great concurrency problem)*
3. **Medical records & visits**: file uploads (prescriptions, x-rays → object storage /
   MinIO).
4. **Notifications**: async reminders via events → message queue.
   *(eventual consistency)*
5. **Billing / invoicing**: extract as a microservice.
   *(service boundaries, distributed data)*
6. **Reporting / analytics dashboard**: aggregation, caching.
   *(read models, CQRS-lite)*
7. **Observability + K8s deploy**: the production-readiness phase.

---

## 8. System-Design Concepts to Deliberately Practice

- **Concurrency**: appointment booking race conditions
  (optimistic vs pessimistic locking).
- **Eventual consistency**: notifications, billing.
- **Idempotency**: booking / payment endpoints (dedupe repeated requests).
- **Caching strategies**: cache invalidation for vet availability.
- **Pagination & N+1 avoidance**: medical history lists.
- **CQRS / read models**: analytics.

---

## 9. Top Recommendation

> Start as a **modular monolith**, keep an **ADR log**, and only add complexity
> (microservices, Kafka, K8s) when a concrete feature forces the need. That way each
> piece of "latest tech" is introduced to solve a real problem **you** felt — which is
> how you build genuine system-design intuition instead of cargo-culting.

---

## 10. Next Steps / Backlog Seed

- [ ] Draft domain model + ERD + bounded-context breakdown
- [ ] Define phase-1 Spring Boot package structure (package-by-feature)
- [ ] Set up Spec Kit and generate a minimal Angular FE from the OpenAPI spec (low priority)
- [ ] Set up repo, CI pipeline skeleton, docker-compose (Postgres)
- [ ] Write first ADR: "Why modular monolith to start"
- [ ] Implement Owners/Pets/Vets CRUD + JWT auth (MVP)

