# Pet Clinic — Progress Log

Last updated: 2026-10-19

Working agreement is in [projectObjectives.md](projectObjectives.md), section 0
(I do the work, the assistant reviews and shows alternatives, no prescribing).

---

## Current task

**Notion task 1:** Draft domain model + ERD (v1 scope below).

Planned order:
1. Define core entities (done, see below)
2. Draw a context diagram (actors, system, external systems)
3. Create the ERD from it, then share for review

**Next step:** I create the ERD and share it for review.

---

## Notion

Database: "pet clinic tasks". Tasks created (all TODO):
1. Draft domain model + ERD + bounded-context breakdown
2. Define phase-1 Spring Boot package structure (package-by-feature)
3. Set up repo, CI pipeline skeleton, docker-compose (Postgres)

Remaining backlog (add later, only when asked): first ADR, Owners/Pets/Vets CRUD + JWT MVP,
Spec Kit frontend.

---

## Decisions so far

### Scope (v1)
- Entities: **owner, pet, vet, appointment, notification**
- Out of scope for now: payment/billing, visits, medical records, vaccinations
- Frontend: not a focus, generated later with Spec Kit + LLM from the OpenAPI spec
- Skipping DDD vocabulary. Only grouping by feature (package-by-feature) later.

### Accounts and roles
- `user_account` holds login data and a `role` column: **USER or ADMIN**
- `owner` and `vet` are **profile tables**, each with a `user_account_id` FK
- Whether an account is a vet or owner is decided by whether the profile row exists
- One account can have both an owner and a vet profile (user switches at login)
- Admin = account with role ADMIN (can also have vet/owner profiles)
- Admin capabilities: admin tab with dashboard of all vets and owners, can
  delete/manage them, can see every appointment for each doctor
- No separate role table for now (a `role` column is enough)
- Possible future role: `clinic_admin` (not decided)

---

## Open questions (to decide while drawing the ERD)

- Unique `user_account_id` on `owner` and `vet` (one profile of each type per account)?
- `role` as varchar + check constraint vs. Postgres enum
- Single `role` column vs. `user_role` join table, if roles ever need to combine
  or `clinic_admin` has partial admin powers
- Is `clinic_admin` scoped to a clinic? (matters only if multi-clinic)
- Active profile: JWT carries active profile (switch issues a new token) vs. all
  profiles in the token
- Rule: can a vet be the vet on an appointment for their own pet?
- Deleting vets/owners: soft delete vs. hard delete vs. block when dependents
  exist vs. deactivate
- Audit log for admin actions (optional for MVP)
- Vet availability: working hours table, no availability (any time bookable unless
  conflicting), or pre-generated slots
- Vet specialties: skip, or a plain string column
- Appointment foreign keys: pet + vet only (owner implied through pet)
- Appointment status lifecycle (e.g. scheduled, cancelled, completed, no-show)
- Double-booking protection: pessimistic lock, optimistic lock (`version` column),
  or DB constraint (unique on vet + start, or Postgres exclusion constraint on a
  time range)
- Notification: recipient (`user_account` or owner only), trigger (appointment created,
  reminder N hours before), what it references, status (pending/sent/failed) to
  support async sending later
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
