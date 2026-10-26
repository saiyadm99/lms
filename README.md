# LearnHub — LMS Course Enrollment & Waitlist System

A full-stack LMS Enrollment & Waitlist System built for a Software Engineer Intern technical assessment.

LearnHub allows students to discover published courses, enroll when seats are available, join waitlists when courses are full, and cancel their own enrollments or waitlist positions. Administrators can manage courses, adjust capacities, and toggle course publication status with automatic deterministic FIFO waitlist promotions.

---

## 1. Tech Stack

- **Backend**: Laravel 12 on PHP 8.2+
- **Database**: MySQL / MariaDB (InnoDB engine with transactions and pessimistic row locking)
- **Frontend**: React 19, Vite 8, React Router 8, Tailwind CSS 4, Lucide React
- **API**: RESTful JSON API authenticated with Laravel Sanctum (Bearer tokens)
- **Testing**: PHPUnit / Laravel Feature & Unit test suites

---

## 2. Directory Structure

```
lms-assignment/
├── client/                     # Frontend React application
│   ├── src/                    # Components, pages, layouts, auth context
│   └── vite.config.js          # Vite config with /api dev proxy
├── server/                     # Backend Laravel application
│   ├── app/                    # Controllers, Models, Requests, EnrollmentService
│   ├── database/               # Migrations, seeders, factories
│   └── tests/                  # Automated PHPUnit tests
└── docs/                       # Architecture specs, test reports, and design reference
    ├── architecture.md
    ├── test-plan.md
    └── design-reference.png
```

---

## 3. Prerequisites

Ensure you have the following installed on your machine:
- **PHP**: `^8.2` (with PDO, MySQL, and OpenSSL extensions enabled)
- **Composer**: `^2.8`
- **Node.js**: `^20.0` or `^22.0`
- **pnpm**: `^9.0` or `^12.0`
- **MySQL / MariaDB**: Running on port `3306` (e.g., via XAMPP)

---

## 4. Setup Instructions

### Step 1: Database Setup
Make sure your MySQL / MariaDB server is running. Create a dedicated database named `lms-assignment`:
```sql
CREATE DATABASE `lms-assignment` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Step 2: Backend Setup (`server/`)
1. Open a terminal in the `server/` directory:
   ```bash
   cd server
   ```
2. Verify your `.env` configuration (matches local MySQL credentials):
   ```ini
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=lms-assignment
   DB_USERNAME=root
   DB_PASSWORD=
   ```
3. Run the migrations to build tables and constraints:
   ```bash
   php artisan migrate
   ```
4. Seed the database with demo accounts and sample courses:
   ```bash
   php artisan db:seed
   ```

### Step 3: Frontend Setup (`client/`)
1. Open a terminal in the `client/` directory:
   ```bash
   cd client
   ```
2. Dependencies are already installed in `node_modules/`. To reinstall if needed:
   ```bash
   pnpm install
   ```

---

## 5. Running the Application

Run both applications concurrently in separate terminals:

### Terminal 1: Backend Server
```bash
cd server
php artisan serve
```
*The Laravel API server starts at: `http://127.0.0.1:8000`*

### Terminal 2: Frontend Dev Server
```bash
cd client
pnpm run dev
```
*The Vite React client starts at: `http://localhost:5173`*

Open your browser and navigate to: **`http://localhost:5173`**

---

## 6. Demo Accounts (Development Only)

The database seeder provisions two default accounts for evaluating both user experiences:

| Role | Email | Password | Access & Experience |
| :--- | :--- | :--- | :--- |
| **Administrator** | `admin@example.com` | `password` | Admin Dashboard, Course Management, Capacity Adjustments, Create/Edit Courses |
| **Student** | `student@example.com` | `password` | Course Catalog, Enroll in Courses, Join Waitlists, My Enrollments Dashboard |

> **Tip**: The login page includes quick-fill buttons for both demo accounts so you can sign in with one click without typing credentials.

---

## 7. Running Automated Tests & Code Verification

### Backend Automated Tests
Run the full PHPUnit test suite:
```bash
cd server
php artisan test
```
*Results: 42 tests passed, 155 assertions, zero failures.*

### Frontend Lint & Production Build
```bash
cd client
pnpm run lint
pnpm run build
```
*Results: 0 lint errors, 0 warnings. Production bundle builds cleanly in ~1.3s.*

---

## 8. API Endpoint Summary

### Public Endpoints
| Method | Route | Description | Response Status |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/register` | Register new student account (always student role) | `201 Created` |
| `POST` | `/api/login` | Authenticate user & receive Sanctum Bearer token | `200 OK` |
| `GET` | `/api/courses` | List published courses with calculated availability | `200 OK` |
| `GET` | `/api/courses/{course}` | View published course details (404 if unpublished) | `200 OK` / `404` |

### Authenticated Student Endpoints (`Authorization: Bearer <token>`)
| Method | Route | Description | Response Status |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/me` | Get authenticated user info & role | `200 OK` |
| `POST` | `/api/logout` | Revoke current access token | `200 OK` |
| `GET` | `/api/me/enrollments` | List student's active enrollments and waitlist | `200 OK` |
| `POST` | `/api/courses/{course}/enrollment` | Enroll in course or join waitlist | `201` (new) / `200` (unchanged) |
| `DELETE` | `/api/courses/{course}/enrollment` | Cancel student's own active record | `200 OK` |

### Administrator Endpoints (`admin` middleware required)
| Method | Route | Description | Response Status |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/admin/courses` | List all courses (including unpublished drafts) | `200 OK` |
| `POST` | `/api/admin/courses` | Create new course (title, capacity, is_published) | `201 Created` |
| `PATCH` | `/api/admin/courses/{course}` | Update title, capacity, or publication status | `200 OK` |
| `DELETE` | `/api/admin/courses/{course}` | Delete course and its associated enrollments | `200 OK` |

---

## 9. Example API Requests

### 1. Register a Student
```bash
curl -X POST http://127.0.0.1:8000/api/register \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "name": "Jane Student",
    "email": "jane@example.com",
    "password": "password123",
    "password_confirmation": "password123"
  }'
```

### 2. Login
```bash
curl -X POST http://127.0.0.1:8000/api/login \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "email": "student@example.com",
    "password": "password"
  }'
```

### 3. Enroll or Join Waitlist
```bash
curl -X POST http://127.0.0.1:8000/api/courses/1/enrollment \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Accept: application/json"
```

### 4. Admin Update Course Capacity
```bash
curl -X PATCH http://127.0.0.1:8000/api/admin/courses/1 \
  -H "Authorization: Bearer <ADMIN_TOKEN>" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "capacity": 5
  }'
```

### 5. Admin Delete Course
```bash
curl -X DELETE http://127.0.0.1:8000/api/admin/courses/1 \
  -H "Authorization: Bearer <ADMIN_TOKEN>" \
  -H "Accept: application/json"
```

---

## 10. Key Architecture & Business Decisions

- **Pessimistic Row Locking (`lockForUpdate()`)**: Every seat-altering request locks the course row first inside a short database transaction. Enrolled counts are calculated strictly inside the lock, preventing race conditions and over-allocation.
- **Deterministic FIFO Promotion**: Waitlist promotion orders by `waitlisted_at ASC, id ASC`. When a seat is freed or capacity is increased, the earliest eligible waiter is promoted automatically.
- **Capacity Reductions**: If an admin reduces capacity below current enrolled students, existing enrollments are strictly preserved.
- **Duplicate Prevention**: A unique database constraint on `(course_id, user_id)` guarantees one record per user per course. Repeated requests are idempotent.
- **More Details**: See [docs/architecture.md](docs/architecture.md) and [docs/test-plan.md](docs/test-plan.md).
---

## 11. Known Limitations

- **Fast SQLite Tests**: Tests run against in-memory SQLite for speed and isolation. Real MySQL row-lock contention is not tested within the automated suite.
- **No Email Delivery**: The application does not send any emails. There is no enrollment confirmation, waitlist promotion notification, or password reset functionality implemented.
- **Limited Mobile Responsiveness**: The UI isn't fully responsive yet mainly designed for desktop viewports. Some layouts, especially the admin tables, may not display well on small screens.
- **No Pagination**: The course list returns all records in a single response. This will become slow as the number of courses grows.
- **No Rate Limiting**: The login and register endpoints have no request throttling, making them vulnerable to brute-force attempts.
- **Tokens Do Not Expire**: Sanctum tokens are issued without an expiry time. A compromised token remains valid until the user explicitly logs out.