# Student-Only Attendance Management Portal (MERN + Mongoose + TypeScript)

A full-stack, secure, responsive Student Attendance Management Portal built with MongoDB, Express.js, React, Node.js (MERN stack), TypeScript, Mongoose, Zod, JWT authentication, and Tailwind CSS.

> **Role & Security Specification**: The application exclusively supports two roles: `admin` and `student`. There is **no teacher role**, teacher dashboard, teacher login, or teacher APIs. Public registration for Admin accounts is strictly forbidden; Admin accounts are provisioned via a secure seed script.

---

## Features

### Authentication & Authorization
- **Role-Based Authorization**: Distinct access controls for `admin` and `student`.
- **JWT Authentication**: Tokens signed with issuer, audience, subject, role, and configurable expiration.
- **Secure Password Hashing**: `bcryptjs` with salt round configuration.
- **Account Status Guard**: Inactive accounts are blocked at authentication and middleware level.
- **Safe Responses**: Password hashes and sensitive internal metadata are stripped from API outputs.

### Student Portal (`/student`)
1. **Dashboard Overview**: Summary of student profile (Name, Roll Number, Class Name, Subject).
2. **Attendance Statistics**: Live cards for Total Sessions, Present Count, Absent Count, Late Count, and Overall Percentage.
3. **Progress Bar Visualizer**: Visual indicator based on percentage thresholds (`<75%` warning in red, `<85%` in amber, `≥85%` in emerald).
4. **Attendance History Table**: Session-by-session records with status badges and date-range filtering (`from` and `to`).
5. **Data Protection**: Students are strictly isolated to viewing only their own attendance data.

### Admin Management Interface (`/admin`)
1. **Overview Analytics**: Portal metrics including active classes, total enrolled students, and quick action shortcuts.
2. **Student Account Management**: Create student accounts and profiles, edit student details, filter by class/search, and toggle active/inactive status.
3. **Class & Enrollment Management**: Create academic classes, update details, view enrolled students, and disassociate students while preserving historical attendance records.
4. **Bulk Attendance Marker**: Interactive daily matrix view to select class & date, toggle individual statuses (`present`, `absent`, `late`), or use bulk actions (`All Present`, `All Absent`).
5. **Class Reports**: Aggregate class metrics, recorded session totals, average class percentage, and student-level breakdowns.

---

## Directory Structure

```
student-attendance/
├── client/                     # React + TypeScript + Vite + Tailwind CSS
│   ├── src/
│   │   ├── api/                # Axios instance & API modules
│   │   ├── components/         # Reusable UI (Navbar, Sidebar, Badge, Modal, ProgressBar)
│   │   ├── context/            # AuthContext (session, login, logout)
│   │   ├── layouts/            # Layout shell
│   │   ├── pages/              # Auth, Student, and Admin pages
│   │   ├── routes/             # AppRoutes & ProtectedRoute guard
│   │   ├── types/              # TypeScript interfaces
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── package.json
│   ├── tailwind.config.js
│   └── vite.config.ts
├── server/                     # Node.js + Express + Mongoose + TypeScript
│   ├── src/
│   │   ├── config/             # DB connection & Zod environment parser
│   │   ├── models/             # User, Class, Student, Attendance Mongoose models
│   │   ├── modules/            # Auth, Students, Classes, Attendance controllers & schemas
│   │   ├── middleware/         # Authenticate, Authorize, Validate, ErrorHandler, NotFound
│   │   ├── scripts/            # Seed script (seed-admin.ts)
│   │   ├── tests/              # Integration tests
│   │   ├── utils/              # JWT, bcrypt, AppError helpers
│   │   ├── app.ts              # Express configuration
│   │   └── server.ts           # Server bootstrap & graceful shutdown
│   ├── package.json
│   ├── tsconfig.json
│   └── vitest.config.ts
├── .env.example
└── README.md
```

---

## Mongoose Models & Schemas

1. **User Model**: `_id`, `name`, `email` (unique, normalized), `passwordHash`, `role` (`admin` | `student`), `isActive` (boolean, default true).
2. **Class Model**: `_id`, `name`, `subject`, `isActive` (boolean, default true).
3. **Student Model**: `_id`, `userId` (ref User, unique), `rollNumber`, `classId` (ref Class). Unique compound index on `{ classId: 1, rollNumber: 1 }`.
4. **Attendance Model**: `_id`, `studentId` (ref Student), `classId` (ref Class), `date` (`YYYY-MM-DD`), `status` (`present` | `absent` | `late`), `markedBy` (ref User). Unique compound index on `{ studentId: 1, classId: 1, date: 1 }`.

---

## REST API Endpoints Summary

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| **POST** | `/api/auth/login` | Public | Authenticate student or admin, return signed JWT |
| **POST** | `/api/students` | Admin | Create student account and profile |
| **GET** | `/api/students` | Admin | List students with search, class filter, and pagination |
| **GET** | `/api/students/:studentId` | Admin | Get student details |
| **PATCH** | `/api/students/:studentId` | Admin | Update student profile and user details |
| **PATCH** | `/api/students/:studentId/status` | Admin | Activate or deactivate student account |
| **POST** | `/api/classes` | Admin | Create new class |
| **GET** | `/api/classes` | Admin | List classes with enrolled student counts |
| **GET** | `/api/classes/:classId` | Admin | Get class details and enrolled students |
| **PATCH** | `/api/classes/:classId` | Admin | Update class details or active status |
| **POST** | `/api/classes/:classId/students` | Admin | Enroll student in class |
| **DELETE** | `/api/classes/:classId/students/:studentId` | Admin | Remove student from class (preserves historical attendance) |
| **POST** | `/api/attendance/mark` | Admin | Bulk mark/upsert student attendance for a class and date |
| **GET** | `/api/attendance/class/:classId` | Admin | View class attendance matrix for a date (includes `not_marked`) |
| **GET** | `/api/attendance/student/:studentId` | Admin / Student | Get student attendance summary and history (Ownership enforced) |
| **GET** | `/api/attendance/report/class/:classId` | Admin | Generate aggregated class attendance report |

---

## Local Setup & Development

### Prerequisites
- Node.js (v18+)
- MongoDB daemon running locally on `mongodb://127.0.0.1:27017` or a MongoDB Atlas URI.

### 1. Backend Setup
```bash
cd server
npm install
npm run seed:admin   # Seeds initial admin account (admin@portal.com / AdminPass123!)
npm run dev          # Starts dev server on http://localhost:5000
```

### 2. Frontend Setup
```bash
cd client
npm install
npm run dev          # Starts Vite dev server on http://localhost:5173
```

---

## Automated Testing

Run integration tests (verifying authentication, authorization guards, bulk attendance upserts, duplicate rejection, and percentage calculation policies):

```bash
cd server
npm test
```

---

## Production Build

```bash
# Build backend TypeScript
cd server
npm run build

# Build frontend production bundle
cd client
npm run build
```
