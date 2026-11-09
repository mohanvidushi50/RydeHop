# RydeHop 🚗

**Smart, Student-Focused Ride Sharing for College Commutes**

RydeHop is a student-focused carpooling web platform designed to make daily college commuting more affordable, convenient, and efficient. It connects students travelling along similar routes so they can share rides, coordinate pickup points, and split travel expenses.

Instead of relying on expensive individual transportation, students can find available rides or offer empty seats to others travelling in the same direction.

---

## 📌 Problem Statement

Daily commuting to college can be expensive and inefficient, especially for students who depend on private cabs, autos, or other individual transportation options.

Students travelling along similar routes often have no convenient way to discover one another and coordinate shared rides.

**RydeHop aims to solve this problem by connecting students with compatible routes and schedules, helping them share travel costs and make better use of available vehicle seats.**

---

## 💡 Our Solution

RydeHop provides a centralized platform where verified college students can:

- Offer available seats in their vehicles.
- Discover rides matching their destination and schedule.
- Request a seat in an available ride.
- Coordinate pickup locations and travel timings.
- Agree on a transparent cost-sharing arrangement.
- Connect with other students through a student-focused environment.

The platform focuses on ride coordination and cost-sharing rather than operating as a taxi-booking or commercial ride-dispatch service.

---

## ✨ Key Features

### 1. Student Verification
- College email-based verification.
- Manual verification support where required.
- Student profiles to encourage a trusted community.

### 2. Offer a Ride
- Publish a ride with source and destination.
- Select travel date and departure time.
- Specify available seats.
- Add pickup details and estimated cost-sharing information.

### 3. Find a Ride
- Search for available rides.
- Discover rides based on route and schedule compatibility.
- View pickup and destination information.
- Request an available seat.

### 4. Ride Matching
- Match students travelling in similar directions.
- Consider departure-time compatibility.
- Check seat availability.
- Evaluate pickup distance and route detours.
- Provide understandable matching results.

### 5. Ride Requests and Coordination
- Send and manage seat requests.
- Accept or decline requests.
- Cancel rides or requests.
- Reveal contact information only after ride confirmation.

### 6. Cost Sharing
- Estimate shared travel expenses using relevant cost factors.
- Display suggested individual contributions.
- Allow participants to mutually agree on the final amount.

*RydeHop does not process payments in its initial MVP.*

### 7. Safety and Privacy
- Report or block users.
- Share trip information with confirmed participants.
- Enable location sharing only during an active trip and with user consent.
- Restrict access to sensitive ride and location information.
- Support trip status updates and safety check-ins.

---

## 🛠️ Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React.js, TypeScript |
| Build Tool | Vite |
| Styling | Tailwind CSS |
| Backend | Node.js, Express.js, TypeScript |
| Database | PostgreSQL |
| Geospatial Database | PostGIS |
| Maps | Leaflet |
| Map Data | OpenStreetMap |
| Routing | OSRM (prototype usage) |
| Real-time Communication | WebSockets |
| Version Control | Git and GitHub |
| Testing | Vitest, Supertest, Playwright |

### Zero-Budget Development Approach

RydeHop is being designed with a zero-budget development approach.

- Prefer open-source technologies and locally hosted development tools.
- Avoid paid APIs and subscription-based services.
- Use browser-based geolocation with user permission.
- Begin with curated campus landmarks and manually selected map locations.
- Use public map and routing services only within their permitted usage limits.
- Keep external service integrations replaceable through provider adapters.

**Note:** Free and open-source software does not mean every hosted service is unlimited or guaranteed to remain free. Public map, geocoding, routing, and hosting services have usage policies and operational limitations. Production deployment will require reviewing these constraints.

---

## 🏗️ System Architecture

RydeHop follows a modular monolithic architecture to keep the initial system manageable, maintainable, and suitable for future expansion.

```text
                 STUDENT
                    |
                    v
          React + TypeScript
                    |
                    v
             Express.js API
                    |
       +------------+------------+
       |            |            |
       v            v            v
   Authentication  Ride       Matching
                   Management  Engine
       |            |            |
       +------------+------------+
                    |
                    v
             PostgreSQL
                + PostGIS
                    |
          +---------+---------+
          |                   |
          v                   v
      Ride Data          Geospatial Data
```

External integrations such as maps, routing, and email services are connected through replaceable service adapters.

---

## 🔄 How RydeHop Works

1. A student registers and completes college verification.
2. The student either offers a ride or searches for an existing ride.
3. RydeHop identifies rides that match the student's route and schedule.
4. The student sends a seat request.
5. The ride provider accepts or declines the request.
6. Once accepted, both students can coordinate their journey.
7. The trip progresses through its respective status updates.
8. After completion, students can provide feedback.

---

## 🗺️ Ride Lifecycle

```text
Draft
  |
  v
Open
  |
  v
Requested
  |
  v
Confirmed
  |
  v
In Progress
  |
  v
Completed
```

Additional outcomes include:

- Declined
- Cancelled
- Expired
- No-show
- Disputed

The backend is responsible for validating ride-state transitions and maintaining accurate seat availability.

---

## 🎯 Project Scope

### MVP — Initial Development

The first version focuses on the core ride-sharing experience:

- Student registration and verification.
- Student profiles.
- Ride creation and discovery.
- Route and schedule-based matching.
- Seat requests and confirmations.
- Basic cost-sharing estimates.
- Map-based pickup selection.
- Basic safety and reporting functionality.

### Future Enhancements

Features considered for future versions include:

- Recurring ride schedules.
- Advanced route optimization.
- Enhanced real-time location sharing.
- Push notifications.
- Improved safety and moderation tools.
- Multi-campus support.
- More advanced ride-matching algorithms.

These features are not part of the initial MVP unless explicitly implemented later.

---

## 🔐 Privacy and Security

Student trust and privacy are important design principles of RydeHop.

The platform aims to:

- Restrict ride discovery and participation to eligible students.
- Avoid exposing exact pickup locations publicly.
- Reveal contact information only after confirmation.
- Require explicit consent for location sharing.
- Limit location sharing to active trips.
- Restrict trip information to authorized participants.
- Provide reporting and blocking mechanisms.
- Validate seat reservations and ride status transitions on the server.

---

## 🚀 Project Status

<kbd>In Development</kbd>

RydeHop is currently in the planning and development stage. Its initial focus is building a functional web-based MVP with reliable ride discovery, matching, and coordination.

---

## 🤝 Contributing

Contributions, suggestions, and feedback are welcome.

If you would like to contribute:

1. Fork the repository.
2. Create a new feature branch.
3. Make your changes.
4. Test your implementation.
5. Submit a pull request with a clear description.

---

## 📄 License

A project license will be added once the repository's distribution and contribution terms have been finalized.

---

**RydeHop — Share the Ride. Split the Cost. Make the Commute Better.**
