# Pet Clinic — Progress Log

Last updated: 2026-11-22

Working agreement is in [projectObjectives.md](projectObjectives.md), section 0
(I do the work, the assistant reviews and shows alternatives, no prescribing).

---

## Current task

**Notion task 2:** Define phase-1 Spring Boot package structure (package-by-feature). In progress.

**Done:** Notion task 1, domain model + ERD (v1 scope below).

ERD content (v1):
- `user`: `email`, `password`, `role`
- `owner`: 1:1 with `user`
- `vet`: 1:1 with `user`
- `pet`: N:1 `owner` (owner 1:N pet)
- `appointment`: N:1 `pet` (pet 1:N appointment) and N:1 `vet` (vet 1:N appointment)

---

## Notion

Database: "pet clinic tasks" (tasks as of 2026-11-22):
1. Draft domain model + ERD + bounded-context breakdown: **Done**
2. Define phase-1 Spring Boot package structure (package-by-feature): **In Progress**

Remaining backlog (add later, only when asked): first ADR, Owners/Pets/Vets CRUD + JWT MVP,
Spec Kit frontend.

---

## Decisions so far

### Scope (v1) — start small
- Entities: **user** (`email`, `password`, `role`), **owner**, **vet**, **pet**, **appointment**
- Relationships: user 1:1 owner, user 1:1 vet, owner 1:N pet, pet 1:N appointment,
  vet 1:N appointment
- **Notifications removed from v1** (moved to later phase)
- Out of scope for now: payment/billing, visits, medical records, vaccinations
- Frontend: not a focus, generated later with Spec Kit + LLM from the OpenAPI spec
- Skipping DDD vocabulary. Only grouping by feature (package-by-feature) later.

### Accounts and roles (updated 2026-11-22)
Three roles:
- **ADMIN** = the doctor (vet): sees their own appointments
- **PATIENT** = a person with their pet(s)
- **OWNER** = the clinic owner: adds/removes vets, assigns vets (exact meaning of
  "assign" still to pin down, see open questions)

- `user` holds login data (`email`, `password`) and a `role` column
- `owner` and `vet` are profile tables, each 1:1 with `user`
- No separate role table for now (a `role` column is enough)
- Replaces the earlier USER/ADMIN model and the "admin dashboard of all vets and
  owners" idea; that capability now belongs to the clinic OWNER

---

## Open questions (to decide while drawing the ERD)

- "Owner assigns vets": assign vets to what? (patients, appointments, a clinic/role?)
- Can one account hold more than one role (e.g. a vet who is also a patient)? If
  not, a single `role` column is enough
- Naming clash: role OWNER (clinic owner) vs. `owner` entity (pet owner). Rename one?
- Unique `user_id` on `owner` and `vet` (enforces the 1:1)?
- `role` as varchar + check constraint vs. Postgres enum
- Rule: can a vet be the vet on an appointment for their own pet?
- Deleting vets/patients: soft delete vs. hard delete vs. block when dependents
  exist vs. deactivate (matters when the owner removes a vet with appointments)
- Vet availability: working hours table, no availability (any time bookable unless
  conflicting), or pre-generated slots
- Vet specialties: skip, or a plain string column
- Appointment foreign keys: pet + vet only (owner implied through pet)
- Appointment status lifecycle (e.g. scheduled, cancelled, completed, no-show)
- Double-booking protection: pessimistic lock, optimistic lock (`version` column),
  or DB constraint (unique on vet + start, or Postgres exclusion constraint on a
  time range)
- Cross-cutting ERD choices: ID type (UUID vs. bigint), time handling (UTC),
  audit columns (`created_at`, `updated_at`), enum vs. lookup table for statuses

---

## Concepts clarified

- **Double-booking:** two concurrent requests both pass the "is the vet free?"
  check and both insert (check-then-insert race condition)
- **Async work:** doing something (e.g. sending a notification) outside the request
  that triggered it, via a background job or message queue (Phase 2)
- **Party model:** a generic person/organization/role structure, considered and
  rejected as over-engineering for this project
- **Profile vs. role:** roles live on `user_account`, vet/owner come from
  the profile rows, so each fact is stored once
