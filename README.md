# QR Code-Based Student Attendance System

A full-stack attendance management application with separate **Admin
Dashboard** and **Student App** frontends, an **Express.js REST API**,
and **MongoDB** for storing student, teacher, and attendance records.

Repository: https://github.com/VarunSahu05/Attendance_System

------------------------------------------------------------------------

## Table of Contents

-   [Project Overview](#project-overview)
-   [Features](#features)
-   [Technology Stack](#technology-stack)
-   [Architecture](#architecture)
-   [Project Structure](#project-structure)
-   [How the Application Works](#how-the-application-works)
-   [Data Models](#data-models)
-   [API Reference](#api-reference)
-   [Prerequisites](#prerequisites)
-   [Local Setup Guide](#local-setup-guide)
-   [Environment Variables](#environment-variables)
-   [Running the Applications](#running-the-applications)
-   [Production Deployment Notes](#production-deployment-notes)
-   [Troubleshooting](#troubleshooting)
-   [Security Notes and Current
    Limitations](#security-notes-and-current-limitations)
-   [Possible Improvements](#possible-improvements)

## Project Overview

The application helps an institution manage student and teacher records
and record attendance using a QR-code session.

It contains three independently runnable components:

1.  **Admin Dashboard** --- an interface for registering teachers and
    students, viewing their lists, creating attendance sessions, and
    reviewing attendance logs.
2.  **Student App** --- an interface for students to verify their roll
    number, scan a session QR code, confirm attendance, and view
    attendance records.
3.  **Backend API** --- an Express server that exposes REST endpoints
    and uses Mongoose to read and write MongoDB records.

The project is divided into three folders so each frontend and the
backend can be installed, configured, run, and deployed separately.

## Features

### Admin Dashboard

-   **Admin login interface:** The current implementation checks a
    username and password in frontend code.
-   **Student registration:** Enter a student name and department. The
    UI generates a roll number in the form `DEPARTMENT-NN`.
-   **Student directory:** View registered students, sort them by roll
    number, filter by department, and delete a student.
-   **Teacher registration:** Enter a teacher name, numeric ID, and
    subject. The UI formats the teacher ID as `T-NN`.
-   **Teacher directory:** View teacher names, IDs, and subjects and
    delete a teacher.
-   **Create attendance session:** Verify a teacher ID, obtain the
    teacher's registered subject, select a department, and generate a
    session ID.
-   **QR code display:** Generate a QR code containing the teacher ID,
    subject, session ID, department, date, and timestamp.
-   **Session progress:** Poll the API for the attendance count every
    five seconds. The session is set to expire after three minutes, and
    the QR code is refreshed when the observed attendance count
    increases.
-   **Attendance log:** View attendance records and filter the displayed
    logs by department and date.

### Student App

-   **Roll-number verification:** Students enter their roll number; the
    backend checks whether that roll number exists.
-   **Student dashboard:** Provides navigation to attendance marking and
    attendance history after the app's login state is set.
-   **QR scanner:** Uses `html5-qrcode` to read the session QR code
    using a camera.
-   **QR data checks:** The frontend checks for the expected teacher ID,
    subject, session ID, and department fields.
-   **Department validation:** The app compares the student's department
    with the department encoded in the QR data.
-   **Teacher lookup:** Retrieves the teacher record to display the
    teacher's name before confirmation.
-   **Duplicate check:** Checks whether an attendance record exists for
    the student, session, and date before sending the mark request.
-   **Attendance history:** Displays the student's saved attendance
    records, ordered by timestamp.

### Backend API

-   REST endpoints built with Express.
-   MongoDB connection through Mongoose.
-   Separate Mongoose models for students, teachers, and attendance
    records.
-   JSON request parsing and CORS configuration for the two deployed
    frontend origins currently listed in the server code.

## Technology Stack

  -----------------------------------------------------------------------
  Area                                Technologies
  ----------------------------------- -----------------------------------
  Admin frontend                      React, Vite, React Router,
                                      Bootstrap dependency, Axios
                                      dependency

  Student frontend                    React, Vite, React Router,
                                      `html5-qrcode`, Axios dependency

  Backend                             Node.js, Express 5

  Database                            MongoDB, Mongoose

  QR scanning                         `html5-qrcode`

  QR generation                       QR image service at
                                      `api.qrserver.com`

  Frontend styling                    CSS

  Configuration                       `.env` files and Vite environment
                                      variables
  -----------------------------------------------------------------------

> Package presence does not necessarily mean every dependency is used in
> the current source. For example, the QR image is currently generated
> through an external QR image URL in the admin session page.

## Architecture

``` text
                   +---------------------------+
                   |       Admin Dashboard     |
                   | React + Vite              |
                   | Students / Teachers       |
                   | Sessions / Attendance Log |
                   +-------------+-------------+
                                 |
                                 | HTTP REST requests
                                 v
                   +---------------------------+
                   |       Express API         |
                   | JSON middleware + CORS    |
                   | Student / Teacher routes  |
                   | Attendance routes         |
                   +-------------+-------------+
                                 |
                                 | Mongoose
                                 v
                   +---------------------------+
                   |         MongoDB           |
                   | Students collection       |
                   | Teachers collection       |
                   | Attendances collection    |
                   +---------------------------+
                                 ^
                                 |
                                 | HTTP REST requests
                   +-------------+-------------+
                   |         Student App       |
                   | React + Vite              |
                   | QR scanning / confirmation|
                   | Attendance history        |
                   +---------------------------+
```

### Request flow

1.  The admin frontend sends a request to the backend using
    `VITE_BACKEND_URL`.
2.  Express routes validate required request fields where implemented.
3.  Mongoose reads or writes the relevant MongoDB model.
4.  The backend responds with JSON.
5.  The frontend updates its interface based on the response.

## Project Structure

``` text
Attendance_System/
├── admin-dashboard/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── LoginForm.jsx
│   │   │   └── Navbar.jsx
│   │   ├── pages/
│   │   │   ├── Home.jsx
│   │   │   ├── RegisterStudent.jsx
│   │   │   ├── StudentsList.jsx
│   │   │   ├── RegisterTeacher.jsx
│   │   │   ├── TeacherRegistration.jsx
│   │   │   ├── TeachersList.jsx
│   │   │   ├── CreateSession.jsx
│   │   │   └── AttendanceLog.jsx
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css
│   ├── .env
│   ├── package.json
│   └── vite.config.js
├── server/
│   ├── config/
│   │   └── db.js
│   ├── models/
│   │   ├── Attendance.js
│   │   ├── Student.js
│   │   └── Teacher.js
│   ├── routes/
│   │   ├── attendanceRoutes.js
│   │   ├── studentRoutes.js
│   │   └── TeacherRoutes.js
│   ├── server.js
│   ├── .env
│   └── package.json
└── student-app/
    ├── public/
    ├── src/
    │   ├── components/
    │   │   ├── Navbar.jsx
    │   │   └── StudentLogin.jsx
    │   ├── pages/
    │   │   ├── Dashboard.jsx
    │   │   ├── mark.jsx
    │   │   └── view.jsx
    │   ├── App.jsx
    │   ├── main.jsx
    │   └── index.css
    ├── .env
    ├── package.json
    └── vite.config.js
```

Some styling files and standard Vite configuration files are omitted
from this overview to keep it readable.

## How the Application Works

### 1. Register students

1.  An admin opens **Register Student**.
2.  The admin enters a name and selects CSE, ECE, or IT.
3.  The frontend generates a roll number such as `CSE-07`.
4.  The frontend sends the name, department, and roll number to
    `POST /api/students/register`.
5.  The backend saves the student document in MongoDB.

The current UI generates a random number from 01 to 50 for the selected
department. Because it does not first check whether the generated roll
number is already taken, a duplicate can cause a database error.

### 2. Register teachers

1.  An admin opens the teacher registration page.
2.  The admin enters a teacher name, numeric ID, and subject.
3.  The frontend formats the ID as `T-01`, `T-02`, and so on.
4.  The frontend sends the teacher data to
    `POST /api/teachers/register`.
5.  The backend checks whether the teacher ID already exists and saves
    the teacher.

### 3. Create an attendance session

1.  The admin enters a registered teacher ID and chooses a department.
2.  The frontend calls the teacher verification endpoint.
3.  If the teacher exists, the app uses the teacher's registered
    subject.
4.  The app creates a session ID using the department and a random
    five-digit number.
5.  It requests the number of students registered in the selected
    department.
6.  It constructs QR data containing the teacher ID, subject, session
    ID, department, date, and timestamp.
7.  The QR image is obtained from an external QR image service.
8.  The page polls the attendance-count endpoint every five seconds and
    ends the session after three minutes. It also attempts to refresh
    the QR after the count changes.

### 4. Scan and confirm attendance

1.  The student logs in using a registered roll number.
2.  The student opens **Mark Attendance** and starts the camera scanner.
3.  The scanner reads the QR content and parses it as JSON.
4.  The frontend checks for required QR fields and compares the
    student's department with the QR department.
5.  The app fetches teacher details using the teacher ID.
6.  The student reviews the displayed details and confirms.
7.  The frontend checks whether an attendance record exists for the
    student, session, and date.
8.  If no existing record is reported, the frontend sends the attendance
    details to the mark endpoint.
9.  The backend saves the attendance document and returns a response.

**Implementation note:** The QR's three-minute expiry is currently
enforced by the admin page's timer. The backend does not independently
validate the QR timestamp or session expiry. A QR payload should not be
treated as proof of authorization on its own.

### 5. View attendance

-   Students request records for their roll number and see the date and
    time of each record.
-   Admins open the attendance log to see records and filter the
    displayed results by department and date.
-   The session page uses the count endpoint to track the number of
    records associated with a session ID.

## Data Models

The following fields are based on the current Mongoose schemas.

### Student

  Field          Type     Rules
  -------------- -------- ------------------
  `name`         String   Required
  `department`   String   Required
  `roll`         String   Required, unique

### Teacher

  Field         Type     Rules
  ------------- -------- -------------------------------------------
  `name`        String   Required, maximum length 10 in the schema
  `teacherId`   String   Required, unique
  `subject`     String   Required, maximum length 10 in the schema

The frontend permits longer teacher names and subjects than the backend
schema, so some submissions may fail validation.

### Attendance

  Field           Type     Rules
  --------------- -------- ----------------------------------------------
  `studentName`   String   Required
  `studentRoll`   String   Required
  `department`    String   Required
  `teacherName`   String   Required
  `teacherId`     String   Required
  `subject`       String   Required
  `sessionId`     String   Required
  `timestamp`     String   Defaults to `Date.now` in the current schema
  `date`          String   Required

The attendance schema does not currently declare a compound unique index
for student, session, and date. The application attempts duplicate
checks in route logic, but the current backend logic should be corrected
before relying on it for duplicate prevention.

## API Reference

Base URL for local development: `http://localhost:5000`

All endpoints below are relative to the backend base URL. Request bodies
should be JSON where applicable.

### Student endpoints

  -----------------------------------------------------------------------------------
  Method                  Endpoint                            Purpose
  ----------------------- ----------------------------------- -----------------------
  `POST`                  `/api/students/register`            Register a student with
                                                              `name`, `department`,
                                                              and `roll`

  `GET`                   `/api/students`                     Get all students

  `GET`                   `/api/students/verify/:roll`        Check whether a roll
                                                              number exists; returns
                                                              a `valid` boolean

  `GET`                   `/api/students/:roll`               Get a student by roll
                                                              number

  `GET`                   `/api/students/count/:department`   Count students in a
                                                              department

  `DELETE`                `/api/students/:id`                 Delete a student by
                                                              MongoDB document ID
  -----------------------------------------------------------------------------------

Example registration body:

``` json
{
  "name": "Asha",
  "department": "CSE",
  "roll": "CSE-07"
}
```

### Teacher endpoints

  -----------------------------------------------------------------------------------
  Method                  Endpoint                            Purpose
  ----------------------- ----------------------------------- -----------------------
  `POST`                  `/api/teachers/register`            Register a teacher with
                                                              `name`, `teacherId`,
                                                              and `subject`

  `GET`                   `/api/teachers`                     Get all teachers

  `GET`                   `/api/teachers/verify/:teacherId`   Check whether a teacher
                                                              exists and return the
                                                              subject

  `GET`                   `/api/teachers/id/:id`              Get a teacher by
                                                              teacher ID

  `DELETE`                `/api/teachers/:id`                 Delete a teacher by
                                                              MongoDB document ID
  -----------------------------------------------------------------------------------

Example registration body:

``` json
{
  "name": "Ravi",
  "teacherId": "T-01",
  "subject": "Networks"
}
```

### Attendance endpoints

  ------------------------------------------------------------------------------------------------
  Method                  Endpoint                                         Purpose
  ----------------------- ------------------------------------------------ -----------------------
  `POST`                  `/api/attendance/mark`                           Submit an attendance
                                                                           record

  `POST`                  `/api/attendance/check`                          Check for a record by
                                                                           `studentRoll`,
                                                                           `sessionId`, and `date`

  `GET`                   `/api/attendance/logs`                           Get attendance logs,
                                                                           sorted by timestamp

  `GET`                   `/api/attendance/student/:roll`                  Get attendance records
                                                                           for a student

  `GET`                   `/api/attendance/count/:sessionId`               Count records for a
                                                                           session ID

  `GET`                   `/api/attendance/attendance?studentRoll=:roll`   Get all attendance
                                                                           records, optionally
                                                                           filtered by roll number
  ------------------------------------------------------------------------------------------------

Example duplicate-check body:

``` json
{
  "studentRoll": "CSE-07",
  "sessionId": "CSE-12345",
  "date": "2026-10-23"
}
```

Example attendance-mark body:

``` json
{
  "studentName": "Asha",
  "studentRoll": "CSE-07",
  "department": "CSE",
  "teacherName": "Ravi",
  "teacherId": "T-01",
  "subject": "Networks",
  "sessionId": "CSE-12345",
  "timestamp": "2026-10-23T09:30:00.000Z",
  "date": "2026-10-23"
}
```

These examples illustrate the expected request shape; use valid records
from your own database when testing.

## Prerequisites

Install the following before starting:

-   **Node.js and npm:** Install a supported Node.js LTS version from
    https://nodejs.org/.
-   **MongoDB:** Use a local MongoDB server or create a MongoDB Atlas
    database at https://www.mongodb.com/atlas.
-   **Git:** Needed to clone the repository.
-   **Camera access:** Needed to test QR scanning in a browser. Browsers
    generally require HTTPS in deployed environments; `localhost` is
    normally treated as a secure development origin.

Check your installed versions:

``` bash
node --version
npm --version
git --version
```

## Local Setup Guide

### Step 1: Clone the repository

``` bash
git clone https://github.com/VarunSahu05/Attendance_System.git
cd Attendance_System
```

### Step 2: Configure MongoDB

1.  Create a MongoDB database or use an existing development database.
2.  Copy its connection string.
3.  Ensure the database user has the required access.
4.  For MongoDB Atlas, configure Network Access to allow connections
    from your development machine. Avoid opening access to all IP
    addresses unless you understand the risk and have an appropriate
    security policy.

### Step 3: Configure the backend

Open a terminal in the repository root:

``` bash
cd server
npm install
```

Create a file named `.env` inside `server/`:

``` env
MONGO_URI=mongodb_connection_string_goes_here
PORT=5000
```

Replace the placeholder with your own MongoDB connection string. Do not
put real credentials in this README or commit the `.env` file.

Start the backend:

``` bash
npm start
```

The server should log that it is running and that MongoDB connected
successfully. If the database connection fails, check `MONGO_URI`,
network access, and database credentials.

### Step 4: Configure the backend CORS allowlist for local development

The current `server.js` allowlist contains the two deployed frontend
origins. Local Vite origins are not included in that list by default, so
browser requests from local frontends may be blocked.

Update the `origin` array in `server/server.js` to include the exact
local origin(s) you use, for example:

``` js
origin: [
  'http://localhost:5173',
  'http://localhost:5174',
  'https://admin-dashboard-omega-dun-12.vercel.app',
  'https://student-app-six-rho.vercel.app'
]
```

Only keep the origins you actually use. Restart the backend after
changing the code.

### Step 5: Configure the Admin Dashboard

Open a second terminal from the repository root:

``` bash
cd admin-dashboard
npm install
```

Create `admin-dashboard/.env`:

``` env
VITE_BACKEND_URL=http://localhost:5000
```

This variable is used by the frontend to construct backend API URLs.
Vite exposes variables prefixed with `VITE_` to client-side code, so
never put private keys, database credentials, or server secrets in this
file.

Start the admin frontend:

``` bash
npm run dev
```

Vite prints the local URL in the terminal, typically
`http://localhost:5173`.

### Step 6: Configure the Student App

Open a third terminal from the repository root:

``` bash
cd student-app
npm install
```

Create `student-app/.env`:

``` env
VITE_BACKEND_URL=http://localhost:5000
```

Start the student frontend:

``` bash
npm run dev
```

Vite will print its local URL. If port 5173 is already occupied by the
admin dashboard, Vite may select another port such as 5174. Add the
actual origin to the backend CORS allowlist.

### Step 7: Test the end-to-end workflow

1.  Confirm MongoDB is reachable and the backend is running.
2.  Open the Admin Dashboard URL.
3.  Log in using the credentials configured in the current frontend
    code. **The current source contains a hardcoded demo credential
    check; replace it with proper server-side authentication before
    using this application in a real environment.**
4.  Register at least one teacher and one student in the same
    department.
5.  Open **Create Session**, enter the teacher ID, select the student's
    department, and generate a session.
6.  Open the Student App in a browser with camera access.
7.  Log in using the registered student's roll number.
8.  Open **Mark Attendance**, scan the QR code, review the details, and
    confirm.
9.  Open the admin attendance log and the student's attendance view to
    check the saved record.

### Step 8: Build the frontends

To check whether the frontends compile for production, run these
commands separately.

Admin:

``` bash
cd admin-dashboard
npm run build
```

Student:

``` bash
cd student-app
npm run build
```

Vite writes production build output to each frontend's `dist/`
directory. The backend's `package.json` defines `npm start` as
`node server.js`.

## Environment Variables

  ------------------------------------------------------------------------
  Variable                Location                 Purpose
  ----------------------- ------------------------ -----------------------
  `MONGO_URI`             `server/.env`            MongoDB connection
                                                   string

  `PORT`                  `server/.env`            Backend port; defaults
                                                   to `5000` if omitted

  `VITE_BACKEND_URL`      `admin-dashboard/.env`   Backend base URL for
                                                   admin requests

  `VITE_BACKEND_URL`      `student-app/.env`       Backend base URL for
                                                   student requests
  ------------------------------------------------------------------------

For local development, set both frontend variables to
`http://localhost:5000`. In production, set each frontend's
`VITE_BACKEND_URL` to the deployed backend URL and rebuild/redeploy the
frontend.

Vite environment variables are embedded into the browser bundle during
build. Treat every `VITE_` variable as public.

## Running the Applications

You need three terminals during local development:

  Terminal   Directory            Command
  ---------- -------------------- ---------------
  Backend    `server/`            `npm start`
  Admin      `admin-dashboard/`   `npm run dev`
  Student    `student-app/`       `npm run dev`

Stop a running development server with `Ctrl+C`.

## Production Deployment Notes

The code currently allows these frontend origins in the backend CORS
configuration:

-   `https://admin-dashboard-omega-dun-12.vercel.app`
-   `https://student-app-six-rho.vercel.app`

If your deployment URLs change, update the CORS allowlist and redeploy
the backend. Configure `MONGO_URI` and `PORT` in the backend hosting
provider's environment settings. Configure `VITE_BACKEND_URL` in the
environment settings for each frontend and redeploy the frontends after
changing it.

The backend must be publicly reachable over HTTPS for deployed frontend
applications. Configure the database network access and credentials for
the deployed server. Do not assume a local `.env` file is uploaded to
the hosting provider.

## Troubleshooting

### Frontend reports a network or CORS error

-   Confirm the backend is running and reachable.
-   Check that the frontend's `VITE_BACKEND_URL` points to the correct
    backend.
-   Check that the exact frontend origin is included in `server.js`.
-   Restart the backend and restart/rebuild the frontend after
    environment changes.

### MongoDB connection fails

-   Check the `MONGO_URI` value and URL-encode special characters in the
    database password when required.
-   Confirm the database user has access to the selected database.
-   Check the database provider's network allowlist.

### QR scanner cannot open the camera

-   Grant camera permission to the browser.
-   Use `localhost` during development or HTTPS in deployment.
-   Check whether another application is using the camera.

### Student or teacher registration fails

-   Check required fields and database connectivity.
-   Teacher IDs and student roll numbers are unique in their respective
    schemas.
-   The teacher schema limits `name` and `subject` to 10 characters even
    though the frontend accepts longer values.

### QR is accepted but attendance does not appear

-   Check the browser console and backend terminal for errors.
-   Verify the QR's department matches the student's registered
    department.
-   Confirm that the session ID, roll number, and date are being sent
    correctly.
-   Review the duplicate-check and mark endpoints; the current backend
    has a duplicate-handling issue that should be fixed before
    production use.

## Security Notes and Current Limitations

This README describes the implementation found in the supplied source
code. The project should be treated as a learning/demo application until
the following items are addressed.

1.  **Hardcoded admin credentials:** The admin login compares
    credentials in frontend code (`admin` / `123`). Anyone who inspects
    the bundle can discover or bypass this check. Move authentication to
    the backend, hash passwords, and protect admin API routes.
2.  **Student login is roll-number verification only:** It verifies that
    a roll number exists; it does not authenticate the student's
    identity with a password or another secure factor.
3.  **API routes are not protected by role-based authorization:**
    Registration, listing, deletion, attendance logs, and other
    operations are reachable through API routes without authentication
    middleware in the current server code.
4.  **QR validation is mainly client-side:** The backend does not verify
    QR signatures, session expiry, or session ownership. A user could
    forge a QR payload or send a direct HTTP request.
5.  **Duplicate attendance logic needs correction:** The mark route uses
    `findOneAndDelete` to look for an existing record. If it finds one,
    it deletes the existing record and returns an error instead of
    preserving it. Replace this with a non-destructive check and enforce
    a compound unique database index.
6.  **Session-count ambiguity:** The current session count endpoint
    counts attendance records by `sessionId`, while the admin page
    compares that count with the total number of students in a
    department. Records from different dates or contexts with a reused
    session ID could affect the count.
7.  **QR image is generated by an external service:** The QR content is
    sent as a URL parameter to `api.qrserver.com`. Avoid putting
    sensitive data in the QR payload and consider generating QR codes
    locally.
8.  **Environment files:** Keep `.env` files out of Git. If any real
    database credential or secret was ever committed, rotate it and
    remove it from repository history as appropriate. A clean secret
    scan does not guarantee the absence of all security problems.
9.  **CORS is not authentication:** CORS controls which browser origins
    can read responses; it does not stop direct API requests from
    scripts or other clients.
10. **Input validation and error handling:** Add consistent validation,
    rate limiting, safer logging, and production error handling.

## Possible Improvements

-   Implement backend authentication and role-based authorization for
    admin, teacher, and student actions.
-   Store passwords only as secure password hashes if password-based
    authentication is introduced.
-   Generate cryptographically random, signed, short-lived QR tokens and
    validate them on the server.
-   Enforce one attendance record per student/session/date using a
    compound unique index.
-   Fix the duplicate-mark route so it never deletes a valid existing
    attendance record.
-   Use a database-backed session model with explicit start and expiry
    timestamps.
-   Generate QR codes locally rather than sending payloads to a
    third-party QR service.
-   Add automated tests for registration, session creation, QR
    validation, duplicate attendance, and authorization.
-   Add pagination and search to large student, teacher, and attendance
    tables.
-   Add a documented `.env.example` file with placeholders only.
-   Add a project license if you want others to reuse the code.

------------------------------------------------------------------------

## Quick Setup Checklist

-   [ ] Install Node.js, npm, Git, and configure MongoDB.
-   [ ] Clone the repository.
-   [ ] Install dependencies in `server/`, `admin-dashboard/`, and
    `student-app/`.
-   [ ] Create the three local `.env` files with your own configuration.
-   [ ] Add your local frontend origins to the backend CORS allowlist.
-   [ ] Start the backend, admin dashboard, and student app in separate
    terminals.
-   [ ] Register a teacher and a student in the same department.
-   [ ] Generate a QR session, scan it, and verify attendance records.
-   [ ] Keep real secrets out of Git and address the security
    limitations before production use.
