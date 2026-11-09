# RydeHop — Product and System Design Specification

**Version:** 1.0  
**Date:** 10 November 2026  
**Product:** Student-only carpooling for predictable college commutes

## 1. Product definition

RydeHop helps verified students going in a similar direction share a car ride and divide an agreed trip cost. It is a coordination and cost-sharing product, not a taxi dispatch service. A student can publish a planned ride with seats, find a ride, request a seat, and coordinate a pickup point.

**Core promise:** Find a nearby student traveling your way, agree on a practical pickup and cost, and travel with clear identity and safety information.

### Product principles
- **Student trust first:** start within one college community and verify campus affiliation.
- **Plan ahead:** daily commutes are predictable; schedule offers and requests rather than depending on a large live driver pool.
- **Confirm the whole match:** route direction, time, remaining seats, pickup feasibility, and detour all matter.
- **Share cost transparently:** show the calculation before acceptance; avoid surge pricing or opaque fares.
- **Reveal only what is needed:** exact home locations and live location are sensitive.
- **Human agreement remains explicit:** a match suggestion is not a booking until both parties confirm.

## 2. Personas, roles, and MVP scope

One account can act as either or both:
- **Ride provider:** a student with a vehicle and available seats.
- **Ride seeker:** a student seeking a seat.
- **Campus moderator/admin:** verifies edge cases, handles reports, and can suspend accounts.

### MVP (first campus, small cohort)
1. Email/college verification with manual fallback.w
2. Profile with name, photo, college, optional pronouns, emergency contact, and rider preferences.
3. Create one-time offer/request: origin, destination, date, departure window, seats, notes.
4. Map address/place search and draggable pickup/dropoff pins.
5. Match suggestions using direction, time, 1 km pickup proximity, capacity, and estimated detour.
6. Request/accept/decline/cancel and a clearly defined trip status flow.
7. In-app trip chat or masked contact reveal after acceptance.
8. Cost estimate and mutually agreed split.
9. Share trip details, check-in, emergency action, report/block.
10. Basic ratings and admin review queue.

### Defer until usage supports it
Recurring schedules, multi-passenger route insertion, payments/escrow, automated identity-document checks, live driver dispatch, surge pricing, ML ranking, chat bots, multi-campus federation, and microservices. Recurring commute templates can be added after one-time matching works.

## 3. User flows and screens

### Onboarding
1. Welcome → select college → verify campus email or join code.
2. Add display name/photo; accept community rules and location/privacy explanation.
3. Optional vehicle and emergency contact setup.
4. Home shows two primary actions: **Find a ride** and **Offer a ride**.

### Find a ride
1. Enter pickup and destination, select date and departure time/window, seats needed, optional preferences.
2. Map preview confirms pins; allow moving pickup to a safe nearby public landmark.
3. See ranked cards: departure, pickup walk/distance, arrival estimate, detour, seats, cost share, provider profile/verification, cancellation history.
4. Open card → inspect shared trip route and rules → request seat.
5. Provider accepts; seeker confirms; trip becomes confirmed. Reveal exact meeting instructions and contact/chat only now.
6. At pickup, both check in. Provider starts trip; status and optional location sharing are visible to participants.
7. Provider marks arrived; seeker confirms completion; both rate/report.

### Offer a ride
1. Enter origin/destination, departure time/window, available seats, vehicle type, pickup flexibility, and cost basis.
2. Preview route and stops; choose an approximate pickup point if desired.
3. Publish. Matching seekers appear as suggestions; provider reviews detour and cost impact.
4. Accept or decline each request. Seat is reserved atomically only after acceptance.
5. Trip screen provides roster, contact, safety/share controls, status updates and completion.

### Core screens
- Home / commute board
- Find ride form
- Offer ride form
- Map picker and route preview
- Match results and match detail
- Requests / notifications
- Trip room (status, chat, route, participants, share trip, emergency/report)
- Profile / trust and preferences
- Admin verification and report queue

## 4. Trip lifecycle and business rules

Use explicit state transitions on the server:
- **Draft → Open → Requested → Confirmed → In progress → Completed**
- Alternate terminal states: **Declined, Cancelled, Expired, No-show, Disputed**

A request has its own state: pending, accepted, declined, withdrawn, cancelled. Avoid treating a match candidate as a reservation.

Rules:
- A trip is Open only until its departure cutoff or seats are full.
- An acceptance transaction verifies the offer is still open and enough seats remain, then increments reserved seats and creates the accepted booking as one atomic operation.
- Both parties confirm before the booking is final if the provider uses two-sided confirmation; otherwise provider acceptance is final and seeker receives an immediate cancellation window.
- Cancellation reason/time is stored. Establish campus-specific grace windows; don't invent punitive penalties in the initial pilot.
- Trip start/completion actions are idempotent. A no-show can be reported after a configurable grace period and reviewed.
- Keep a state-transition audit log. Never allow clients to set arbitrary status values.

## 5. Matching and ranking

Matching is a two-stage process: cheap candidate discovery, then route-aware scoring. Apply hard constraints before ranking.

### Hard filters
- Same active campus/community and verified users.
- Offer and request are for compatible direction (offer origin near request origin and offer destination near request destination, or pickup/dropoff fall along the offer route).
- Same date; departure times within configurable window (initially ±30 minutes).
- Enough unreserved seats.
- Pickup within the initial 1 km limit; allow a public landmark/pickup adjustment.
- Both accounts are active and neither has blocked the other.
- Detour estimate below configured limits (initial pilot: e.g. <=10 minutes and <=20% route-time increase; tune with students).

### Candidate generation
Prototype: filter in-memory/SQLite records by campus, date/time, open status and bounding box; calculate great-circle pickup distance and direction.
Production: Postgres + PostGIS spatial indexes for accurate meter-based radius checks. Optionally compute H3 cell IDs for coarse bucketing/analytics. Search the point's cell and neighbors (or nearby cells covering the search radius), then apply exact distance and route checks. A geohash/H3 cell is a candidate filter, never proof that two points are within 1 km: cells have edges, and close points may cross cell boundaries. PostGIS `ST_DWithin` with geography uses meters and can use a spatial index.

### Route overlap and detour
Do not use straight-line origin/destination distance as the final route decision. Route overlap may be estimated by:
1. Obtain the provider route polyline.
2. Measure seeker pickup/dropoff points against route segments or sample the polyline at fixed intervals.
3. Require pickup before dropoff in travel direction.
4. Compute provider baseline ETA and ETA with added pickup/dropoff stops.
5. Detour = added ETA and added distance. Present both in minutes/km.

For a very early prototype, use origin/destination proximity + direction and label the result an approximate suggestion. Integrate a routing engine before claiming an accurate detour. Map matching is useful for noisy GPS traces (aligning sampled location points to a plausible road path), not necessary for static planned addresses. OSRM exposes route/table/match services; managed routing APIs can provide traffic-aware durations and matrices.

### Ranking formula (initial explainable heuristic)
For each valid candidate:
- 35% pickup convenience (walk/drive distance to pickup)
- 25% route overlap / added detour
- 20% departure-time difference
- 10% destination proximity
- 10% trust/reliability signals (verified status is a hard gate; use completion/cancellation history carefully)

Convert each feature to 0–1 normalized scores with campus-tuned thresholds. Show the reason for ranking (“pickup 450 m away; 4 min estimated detour; departure 10 min later”) instead of a mysterious score. Never use protected characteristics to rank. Collect feedback before adding predictive models.

### Cost splitting
For MVP, calculate a suggested share from declared trip cost:
- Shared cost pool = estimated fuel + tolls + parking actually shared.
- Per-seat suggestion = shared pool / number of riders sharing (including provider only if the group decides to include the provider).
- Show assumptions and let participants agree before confirmation.
- Do not present the figure as a fare guarantee or allow profit-oriented pricing. Payment processing is out of MVP; record the agreed amount and mark it settled outside RydeHop.
- Production may support receipts and payments after legal, tax, insurance, and campus policy review.

## 6. Geospatial and mapping design

### Geohash, H3, and PostGIS
- **Geohash:** compact string prefix gives hierarchical rectangular cells, useful for simple coarse lookup and sharing location buckets. Prefix queries have boundary false negatives unless neighboring cells are included.
- **H3:** hierarchical hexagonal grid, useful for neighbor rings, aggregation and heat maps. Hex cells reduce directional artifacts but still have approximate geometric containment across hierarchy; do exact geometry after cell lookup.
- **PostGIS:** preferred production source of truth for points, lines, radius and corridor queries. Store coordinates as WGS84 geography when meter semantics are desired; add GiST indexes and use index-aware predicates such as ST_DWithin.
- **Recommendation:** do not add geohash or H3 to the first no-database demo. Use PostGIS for a real pilot. Add H3 only if cell-based analytics or fast candidate partitioning demonstrably helps.

### 1 km pickup rule
Treat 1 km as a product default, not a universal truth. Decide whether it means straight-line separation between the seeker’s requested pickup and the driver route, or actual walking distance to an agreed pickup point. For MVP, use radius distance for candidate discovery, then let both users select an accessible public point. In production, distinguish straight-line radius from network walking distance and display the type.

### Map UX and provider choice
Map SDK handles rendering, geocoding/place search, route polylines and user location. Keep provider calls behind a server adapter so provider can change. Confirm terms, attribution, regional coverage, retention/caching limits and cost before selecting a provider. Do not expose server API keys in React. Alternatives include managed APIs (higher convenience and traffic data, metered cost) or OSM-based routing (more control, but operations, freshness, coverage and licensing responsibilities).

## 7. Architecture: prototype and production

### Stage 0 — demo, no database
React SPA + Express API. Use seeded mock data or in-memory objects; persist nothing sensitive and clearly mark demo accounts/data. Provide deterministic sample matches. A JSON file can support a local walkthrough but is not safe for concurrent writes or real users. Avoid real phone numbers, precise home locations, payments and live location. This stage proves the screens and flow only.

### Stage 1 — campus pilot
React web app / responsive PWA → Express modular monolith → managed PostgreSQL with PostGIS. Modules: auth/campus verification, profiles, offers, requests/bookings, matching, notifications, moderation. Use object storage only for profile photos if necessary. Add email notifications first; WebSocket/SSE for trip room only if polling is not acceptable. A managed provider can supply email/OAuth, but server authorization remains authoritative.

### Stage 2 — growing service
Keep the modular monolith. Add Redis only for rate limits, short-lived match cache, and job coordination; a durable queue/outbox for notifications; background worker for route calls and reminders; push via FCM/APNs/web push. Add read replicas or partition by campus only after observed database pressure. A service boundary or separate matching worker is justified only when independent scaling/ownership requires it.

### Request path
1. React sends validated request with auth token and idempotency key.
2. Express authenticates and authorizes; validates schema and quotas.
3. Service reads/writes PostgreSQL transactionally.
4. Matching service fetches coarse candidates, then requests route estimates through a provider adapter with timeout/cache.
5. Persist offer/request/booking and an outbox event in the same transaction.
6. Worker sends notification; failed delivery retries. Client also fetches canonical state from API.
7. Observability records request ID, latency, error class and non-sensitive matching metrics.

## 8. Data model

Use UUID primary keys, UTC timestamps, explicit campus IDs and soft deletion only where audit needs it. Avoid storing duplicated computed values unless cached with an expiry/version.

- **Campus**(id, name, email_domains, join_code_hash, timezone, policy_url, active)
- **User**(id, campus_id, email, email_verified_at, display_name, photo_key, status, created_at)
- **Profile**(user_id, preferences_json, emergency_contact_encrypted, consent_version)
- **Vehicle**(id, owner_id, type, make_model_optional, color_optional, plate_encrypted_optional, seat_capacity, verified_status)
- **RideOffer**(id, provider_id, campus_id, origin_geog, destination_geog, origin_label_private, destination_label_private, route_polyline, departure_at, time_flex_min, seats_total, seats_reserved, cost_basis, status, expires_at, created_at)
- **RideRequest**(id, seeker_id, campus_id, pickup_geog, dropoff_geog, departure_at, time_flex_min, seats_needed, max_detour_min, status, expires_at)
- **MatchCandidate**(id, offer_id, request_id, score, pickup_distance_m, baseline_eta_s, shared_eta_s, detour_s, explanation_json, computed_at, expires_at)
- **Booking**(id, offer_id, request_id, provider_id, seeker_id, seats, agreed_share_minor, currency, status, confirmed_at, cancelled_at)
- **TripEvent**(id, trip/booking_id, actor_id, event_type, payload_minimized, created_at)
- **Message**(id, booking_id, sender_id, body, created_at, deleted_at)
- **Rating**(id, booking_id, rater_id, rated_id, score, tags, created_at)
- **SafetyReport**(id, reporter_id, reported_user_id, booking_id, category, details_private, status, created_at, resolution)
- **DeviceToken**(id, user_id, token, platform, updated_at)
- **OutboxEvent**(id, event_type, aggregate_id, payload, created_at, delivered_at, attempt_count)

Indexes: B-tree on campus/status/departure; GiST on origin/destination geography; unique constraint preventing duplicate active booking per request/offer as appropriate; partial indexes on open rides; unique idempotency key scoped to user/endpoint. Keep precise location out of logs and analytics. Store route estimates with provider/time/version and expiry.

## 9. API outline

Prefix: `/api/v1`. JSON, ISO-8601 UTC time, consistent error envelope `{ error: { code, message, requestId } }`. Validate request bodies and return field-level validation errors. Auth is bearer token or secure session; never trust userId in body.

### Identity/profile
- `POST /auth/verify-email` — begin or complete campus verification.
- `GET /me`, `PATCH /me` — profile/preferences.
- `POST /me/device-tokens` — register notification endpoint.
- `POST /me/blocklist` — block user.

### Rides and matching
- `POST /ride-offers`, `GET /ride-offers?date=&from=&to=&seats=`, `GET /ride-offers/:id`, `PATCH /ride-offers/:id`, `POST /ride-offers/:id/cancel`
- `POST /ride-requests`, `GET /ride-requests/mine`, `POST /ride-requests/:id/cancel`
- `GET /matches?requestId=` — candidate list with reason, distance type, estimates, freshness.
- `POST /ride-requests/:id/matches/:offerId` — request seats; requires idempotency key.
- `POST /bookings/:id/accept`, `POST /bookings/:id/decline`, `POST /bookings/:id/cancel`, `POST /bookings/:id/confirm`

### Trip/safety
- `GET /trips/:id`, `POST /trips/:id/start`, `POST /trips/:id/arrived`, `POST /trips/:id/complete`
- `POST /trips/:id/share-link` — short-lived, revocable, scoped access.
- `POST /trips/:id/location` — only during active trip, participant authorization, throttled and expiring.
- `POST /trips/:id/reports`, `POST /trips/:id/ratings`
- `GET /trips/:id/messages`, `POST /trips/:id/messages`

### Admin
- `GET /admin/verifications`, `POST /admin/users/:id/verify`
- `GET /admin/reports`, `POST /admin/reports/:id/resolve`, `POST /admin/users/:id/suspend`

Enforce role and object-level checks on every booking/trip/message/report endpoint; test that changing an ID never reveals another student's details.

## 10. Safety, privacy, and abuse controls

- Verify campus email domain or issued invite code; domain verification proves access to an email domain, not identity or driving qualification.
- Display verification level honestly. Vehicle and license checks are a separate future policy decision.
- Hide exact origin until match acceptance where possible; use approximate pins in public listings.
- Collect live GPS only during an explicitly started trip; foreground consent, clear indicator, participant-only access, auto-expiry and deletion policy.
- Offer trusted contact trip sharing and emergency call shortcuts; don't imply emergency services are monitored by RydeHop.
- Report, block, moderation queue, user suspension, cancellation/no-show review and appeal path.
- Avoid public exact route histories, home addresses, license plates and emergency contacts.
- TLS in transit, encryption at rest, least-privilege admin access, secret manager, short-lived tokens, audit events, backups and tested restore.
- Rate limit signup, search, requests, messages, location updates and invite use. Validate coordinates/ranges and message lengths; prevent enumeration.
- Minimize retained GPS and message data; publish retention, consent and deletion practices before pilot.
- Follow OWASP API risks including broken object authorization, authentication, property-level authorization, unrestricted resource consumption and unsafe upstream API consumption; follow Express production security guidance such as TLS, Helmet, safe cookies, input validation and login throttling.

## 11. Reliability and scaling

For the initial cohort, one Express deployment and one managed Postgres database are enough. Set explicit timeouts for geocoding/routing; provider failure should degrade to approximate matches or an honest “route estimate unavailable,” never block cancellation or trip safety actions. Cache route estimates briefly by origin/destination/time bucket where provider terms allow. Use a circuit breaker/backoff for upstream outages and quotas.

Use a transactional outbox for notifications so a committed booking isn't lost if email/push fails. Notifications are hints; clients fetch API state. Make booking writes transactional and idempotent to avoid overbooking on simultaneous acceptances. Use database constraints as final guard. Add structured logs, request IDs, error monitoring, latency and notification-failure alerts, and a restore-tested backup plan.

Initial service goals (pilot targets, not promises): ordinary API p95 under 500 ms excluding route-provider calls; match results under 3 s when routing provider responds; booking mutation p95 under 700 ms; 99.5% monthly availability during pilot. Track match-to-request, acceptance, trip completion, cancellation/no-show, median savings reported, safety reports, and support resolution time. Avoid measuring success only by raw matches.

## 12. Cost and build sequence

### Practical stack
- Frontend: existing React; responsive PWA before native apps.
- Backend: existing Node.js + Express, modular monolith.
- Prototype storage: in-memory/seed data only.
- Pilot database: PostgreSQL + PostGIS.
- Maps/routing: provider adapter; pick provider after coverage, pricing and terms review.
- Notifications: email first, then push through FCM/APNs/web push.
- Hosting: one managed app runtime and managed database in one region near campus.
