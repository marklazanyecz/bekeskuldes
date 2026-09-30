# BékésKüldés

A web application for planning and publishing weekly football referee assignments in Békés county, Hungary.

> **Status:** early setup. This README is a living document and will change as the project and the requirements
> (to be finalised with the Referees' Committee) evolve.

## Table of contents

- [Background](#background)
- [What the application does](#what-the-application-does)
- [Architecture](#architecture)
- [Tech stack](#tech-stack)
- [Repository layout](#repository-layout)
- [Getting started](#getting-started)
- [Roadmap](#roadmap)
- [Open questions](#open-questions)
- [Author](#author)

## Background

Assigning referees to matches is a recurring pain point for county-level Referees' Committees. In Békés county the
process is currently manual:

1. Referees sign up for every match round (*forduló*) through a **Google Form**. For each day from Thursday to
   Wednesday they state until when / from when they can take matches (or that they are unavailable for the whole
   day), their starting settlement, and whether they have a car.
2. At the start of every season each referee also declares which teams they do not want to referee and which
   fellow referees they do not want to work with.
3. After the form deadline (usually the Sunday evening before the round) the county's assigner enters the round's
   matches into an **Excel** sheet, division by division, and assigns referees by hand.

While assigning, the assigner has to weigh several factors:

- **Squad classification (*keret*)**, announced at the start of every half-season. Referees can only be assigned up
  to a certain division.
- **Form during the season**, based on referee observer ratings and press coverage, which decides whether an
  important match can be entrusted to a given referee.
- **Penalties after serious mistakes**, based on observer evaluations, e.g. being assigned to a lower division, a
  suspension of one or more weeks, half a season, or an indefinite suspension.

## What the application does

The first version is a web application usable on both mobile and desktop. The exact feature set may change after
consultation with the Referees' Committee.

There are three user roles.

### Referees' Committee member (*Játékvezetői bizottsági tag*)

Per referee, the member can set:

- squad classification
- observer rating
- penalty and its duration
- whether the referee is eligible for a *reward* assignment
- activation / deactivation of a referee
- validity of the theory test
- validity of the physical test
- validity of the sports medical certificate

### Referee assigner (*Játékvezető küldő*)

The assigner is responsible for planning each round and can configure:

- a match on which people are assigned **manually**
- specific positions of a match that are assigned manually
- whether every position of a match has to be filled
- a match on which **no** referees should be assigned

Manual assignment may deliberately send a referee to a division above their squad classification.

**Core requirement: automatic first-draft assignment.** An algorithm (and possibly AI) takes the per-match settings,
the referees' own settings and the per-referee settings into account and generates a first assignment for every
match that was not marked as manual. The assigner then reviews the draft, adjusts or completes it, and publishes it.

*Possible later extension: integration with the IFA system so that assignments are entered there automatically.*

### Referee (*Játékvezető*)

A referee can set:

- their availability on specific days of a round (possibly for later rounds too, subject to agreement with the
  committee)
- their starting settlement
- whether they have a car
- at the start of every season:
  - which teams they do not want to referee
  - which referees they do not want to work with

## Architecture

```
   Browser ──► Frontend ──────► Keycloak   (login, issues JWT)
                  │
                  │ REST + JWT
                  ▼
   ┌─────────────────────────────────────┐
   │ Backend (Spring Boot)               │
   │                                     │
   │  ┌───────────────────────────────┐  │
   │  │ Calculator (library)          │──┼──► optional AI provider
   │  │ in-process, no HTTP endpoints │  │    (GPT or Gemini)
   │  └───────────────────────────────┘  │
   └──────────────────┬──────────────────┘
                      │
                      ▼
                 PostgreSQL
```

- **Backend** owns the domain data (referees, matches, rounds, availability, assignments), exposes the REST API and
  validates Keycloak JWTs as an OAuth2 resource server.
- **Calculator** is a library that generates the first-draft assignment. It has no HTTP endpoints: the backend calls
  its functions in-process. It is built as a separate artifact that the backend depends on, and it may access the
  database through the backend's data source. Algorithmic and AI-based strategies are meant to live behind a common
  interface; which one is used (or a hybrid) is not decided yet.
- **Keycloak** handles authentication.

This diagram shows the current plan and may change.

## Tech stack

| Area | Technology |
| --- | --- |
| Database | PostgreSQL |
| Backend | Java 25+ or Kotlin 2.4.x (final choice pending), Spring Boot 4.x on Spring Framework 7.x |
| Calculator | Java 25+ or Kotlin 2.4.x, Spring Boot 4.x on Spring Framework 7.x; Spring AI for the AI-based approach |
| Frontend | React 19.x (web); latest React Native possible later |
| Authentication | Keycloak 26.x |
| AI (optional) | GPT or Gemini API, exact model to be chosen after requirements gathering |

Further technology decisions will be made later.

## Repository layout

| Directory | Contents |
| --- | --- |
| [`bekeskuldes-backend`](bekeskuldes-backend) | Main Spring Boot backend and REST API |
| [`bekeskuldes-calculator`](bekeskuldes-calculator) | Assignment generation library, called in-process by the backend |
| [`bekeskuldes-frontend`](bekeskuldes-frontend) | React web frontend |
| [`bekeskuldes-docker`](bekeskuldes-docker) | Docker setup for local infrastructure (PostgreSQL, Keycloak) |

## Getting started

Not available yet. Build and run instructions will be added here once the backend, database and Docker setup exist.

Planned prerequisites:

- JDK 25+
- Docker (for local PostgreSQL and Keycloak)
- Node.js (for the frontend)

## Roadmap

| Date | Milestone |
| --- | --- |
| 2026-09-30 | Finalise consultation and requirements gathering with the Referees' Committee |
| 2026-11-30 | Database, database connection and static pages |
| 2027-01-31 | Referee and Committee member features |
| 2027-02-28 | Referee assigner features and the assignment mechanism |
| 2027-03-31 | Bug fixes, validation of testing |
| 2027-04-30 | Finalise documentation |
| 2027-05-16 | Presentation to the referee community; evaluate and document the planned questionnaire |

## Open questions

- Final backend language: Java or Kotlin.
- Whether the first-draft generation is a pure algorithm, AI-assisted, or a hybrid, and which AI provider and model
  would be used.
- Whether the first release must also work well as a mobile web app, or whether a native app follows later.
- How referee availability for future rounds (beyond the next one) is handled.
- Whether and when to integrate with the IFA system.

## Author

Lazanyecz Márk, BSc Computer Science (Programtervező informatikus), University of Szeged.
