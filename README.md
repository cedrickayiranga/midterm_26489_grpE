# Stadium Management System

A backend REST API for managing stadium events, bookings, and ticketing — with full Rwandan administrative location hierarchy support.

---

## Overview

The Stadium Management System allows customers to browse events and book tickets, staff to manage events, and admins to oversee the entire system. All users are tied to Rwanda's administrative structure (Province → District → Sector → Cell → Village) using a self-referencing Location entity.

---

## Tech Stack

- **Backend:** Spring Boot 4.0.3
- **Database:** PostgreSQL
- **ORM:** JPA / Hibernate
- **Build Tool:** Maven
- **Java Version:** Java 17
- **Testing:** Postman

---

## Project Structure

```
stadiumManagement/
├── src/main/java/auca/ac/rw/stadiumManagement/
│   ├── domain/          # Entities and Enums
│   ├── repository/      # Data access layer (JpaRepository + custom JPQL)
│   ├── service/         # Business logic and validation
│   └── controller/      # REST endpoints (ResponseEntity)
└── README.md
```

---

## Database Schema

The system includes **8 tables** with full Rwanda location hierarchy support.

| Table | Description |
|---|---|
| `location` | Self-referencing hierarchy (Province → District → Sector → Cell → Village) |
| `app_user` | System users — named `app_user` because `user` is a reserved keyword in PostgreSQL |
| `user_profile` | Extended user information — One-to-One with User |
| `stadium` | Stadium details (name, capacity, location string) |
| `event` | Events hosted at a stadium |
| `booking` | User bookings for events — with duplicate prevention |
| `ticket` | Individual tickets linked to a booking and event |
| `user_events` | Many-to-Many join table (User ↔ Event) |

---

## ERD (Entity Relationship Diagram)

<img width="557" height="805" alt="erd diag" src="https://github.com/user-attachments/assets/7fa0307f-3425-4ba5-a0c5-0900db4ea195" />




## Relationships

| Type | Entities | Implementation |
|---|---|---|
| Self-referencing | Location → Location | `@ManyToOne parent` with `parent_id` FK |
| One-to-One | User ↔ UserProfile | `@OneToOne` / `mappedBy="user"` |
| One-to-Many | Stadium → Events | `@OneToMany mappedBy="stadium"` |
| One-to-Many | User → Bookings | `@OneToMany mappedBy="user"` |
| One-to-Many | Booking → Tickets | `@OneToMany mappedBy="booking"` |
| Many-to-Many | User ↔ Event | `@JoinTable` → `user_events` join table |

---

## Enums

| Enum | Values |
|---|---|
| `ELocationType` | `PROVINCE`, `DISTRICT`, `SECTOR`, `CELL`, `VILLAGE` |
| `ERole` | `CUSTOMER`, `STAFF`, `ADMIN` |
| `EBookingStatus` | `PENDING`, `CONFIRMED`, `CANCELLED` |
| `ETicketStatus` | `VALID`, `USED`, `CANCELLED` |

---

## Key Features

- **Location Hierarchy:** Self-referencing Location entity reflecting Rwanda's 5-level administrative divisions
- **Province Queries:** Retrieve all users from a province regardless of their exact location level using custom JPQL traversing up to 5 parent levels
- **Village/Location Queries:** Retrieve users by any location level (Village, Cell, Sector, District, Province) by direct location match
- **Event Management:** Create and manage stadium events with date and time
- **Booking System:** Users can book tickets for events with duplicate prevention via `existsByUserAndEvent()`
- **Ticket Generation:** Tickets are linked to bookings and events
- **User Roles:** `CUSTOMER`, `STAFF`, `ADMIN`
- **Pagination & Sorting:** Supported on users and events via Spring Data `Pageable`
- **Duplicate Prevention:** `existsBy` checks before every save operation

---

## Running the Project

1. Ensure PostgreSQL is running
2. Create the database:
```sql
CREATE DATABASE stadium_db;
```
3. Update `src/main/resources/application.properties`:
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/stadium_db
spring.datasource.username=postgres
spring.datasource.password=your_password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
server.port=8080
```
4. Run:
```bash
mvn spring-boot:run
```
5. Server starts at `http://localhost:8080`

---

## API Endpoints

**Base URL:** `http://localhost:8080`

---

### Location Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/locations/parent` | Create a Province (top-level location) |
| POST | `/api/locations/child?parentCode={code}` | Create District, Sector, Cell, or Village |
| GET | `/api/locations/all` | Get all locations |
| GET | `/api/locations/{id}` | Get location by ID |
| GET | `/api/locations/provinces` | Get all provinces |
| GET | `/api/locations/code/{code}` | Get location by code |
| GET | `/api/locations/parent/{parentCode}` | Get children by parent code |
| PUT | `/api/locations/update/{id}` | Update location |
| DELETE | `/api/locations/delete/{id}` | Delete location |

---

### User Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/users/save` | Create user |
| GET | `/api/users/all` | Get all users |
| GET | `/api/users/{id}` | Get user by ID |
| PUT | `/api/users/update/{id}` | Update user |
| DELETE | `/api/users/delete/{id}` | Delete user |
| GET | `/api/users/by-province-code?code={code}` | Get all users in a province by code |
| GET | `/api/users/by-province-name?name={name}` | Get all users in a province by name |
| GET | `/api/users/by-location-code?code={code}` | Get users by exact location code (any level) |
| GET | `/api/users/by-location-name?name={name}` | Get users by exact location name (any level) |
| GET | `/api/users/paginated?page=0&size=5&sort=name` | Get users paginated and sorted |

---

### User Profile Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/user-profiles/save` | Create user profile |
| GET | `/api/user-profiles/all` | Get all user profiles |
| GET | `/api/user-profiles/{id}` | Get profile by ID |
| GET | `/api/user-profiles/user/{userId}` | Get profile by user ID |
| PUT | `/api/user-profiles/update/{id}` | Update profile |
| DELETE | `/api/user-profiles/delete/{id}` | Delete profile |

---

### Stadium Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/stadiums/save` | Create stadium |
| GET | `/api/stadiums/all` | Get all stadiums |
| GET | `/api/stadiums/{id}` | Get stadium by ID |
| PUT | `/api/stadiums/update/{id}` | Update stadium |
| DELETE | `/api/stadiums/delete/{id}` | Delete stadium |

---

### Event Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/events/save` | Create event |
| GET | `/api/events/all` | Get all events |
| GET | `/api/events/{id}` | Get event by ID |
| PUT | `/api/events/update/{id}` | Update event |
| DELETE | `/api/events/delete/{id}` | Delete event |
| GET | `/api/events/paginated?page=0&size=5&sort=name` | Get events paginated and sorted |
| GET | `/api/events/search?name={name}&page=0&size=5` | Search events by name |

---

### Booking Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/bookings/save` | Create booking |
| GET | `/api/bookings/all` | Get all bookings |
| GET | `/api/bookings/{id}` | Get booking by ID |
| PUT | `/api/bookings/update/{id}` | Update booking |
| DELETE | `/api/bookings/delete/{id}` | Delete booking |
| GET | `/api/bookings/by-user/{userId}` | Get bookings by user |

---

### Ticket Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/tickets/save` | Create ticket |
| GET | `/api/tickets/all` | Get all tickets |
| GET | `/api/tickets/{id}` | Get ticket by ID |
| PUT | `/api/tickets/update/{id}` | Update ticket |
| DELETE | `/api/tickets/delete/{id}` | Delete ticket |
| GET | `/api/tickets/by-booking/{bookingId}` | Get tickets by booking |

---

## API Summary

- **Total Endpoints:** 40+
- **CRUD Operations:** Full CRUD for all 7 entities
- **Query Features:** Province search, location search, pagination, sorting, name search
- **Error Handling:** Descriptive error messages with appropriate HTTP status codes
- **Response Format:** JSON

---

## Recommended Postman Testing Order

Follow this order so all foreign keys exist before they are referenced:

```
1.  POST /api/locations/parent          → Create Province
2.  POST /api/locations/child           → Create District
3.  POST /api/locations/child           → Create Sector
4.  POST /api/locations/child           → Create Cell
5.  POST /api/locations/child           → Create Village
6.  POST /api/users/save                → Create User (use Village ID)
7.  POST /api/user-profiles/save        → Create UserProfile (use User ID)
8.  POST /api/stadiums/save             → Create Stadium
9.  POST /api/events/save               → Create Event (use Stadium ID)
10. POST /api/bookings/save             → Create Booking (use User ID + Event ID)
11. POST /api/tickets/save              → Create Ticket (use Booking ID + Event ID)
12. Test all GET, PUT, DELETE endpoints
13. Test province search, location search, pagination
```

---

## User Roles

| Role | Description |
|---|---|
| `CUSTOMER` | Can browse events and book tickets |
| `STAFF` | Can manage events and view bookings |
| `ADMIN` | Full system access |

---

## Course

**Web Technology and Internet (INSY 8322)** — Adventist University of Central Africa (AUCA)  
Instructor: Patrick DUSHIMIMANA
