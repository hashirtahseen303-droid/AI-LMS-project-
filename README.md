# Nawaz Sharif School of Eminence — LMS Portal

A modular, mobile-responsive LMS (green + white theme) built on **React 19 + TanStack Start + TanStack Router + Tailwind CSS v4 + shadcn/ui**.

> Note on the stack: this project runs on TanStack Start (React) — Node/Express/MongoDB/Mongoose are not part of this runtime. As agreed for v1, all data is persisted **client-side in `localStorage`** through a single data layer (`src/lib/store.ts`) whose shape mirrors the requested Mongoose models, so it can be swapped for a real backend (Lovable Cloud / Postgres, or your own API) without touching the UI.

## Run locally

```bash
npm install     # or: bun install
npm run dev     # starts the dev server on http://localhost:8080
npm run build   # production build
```

No `.env` is required for the demo — everything runs in the browser. When a backend is added, put values in `.env` (e.g. `VITE_API_URL=`) and read them via `import.meta.env`.

## Folder structure

```text
src/
  routes/                   file-based routes (URL = filename)
    __root.tsx              root layout: AuthProvider + Toaster
    index.tsx               "/" -> redirects to dashboard
    login.tsx               sign in (password show/hide)
    register.tsx            self-signup (student / parent, view-only)
    dashboard.tsx           stats + latest notices
    classes.index.tsx       class list + CRUD (admin only)
    classes.$classId.tsx    class dashboard: students, subjects, marks, attendance
    students.tsx            global student directory (search + filter)
    attendance.tsx          pick a class to mark attendance
    marks.tsx               pick a class for the gradebook
    reports.tsx             class performance, top 10, defaulters (<75%)
    notices.tsx             notice board
    my-report.tsx           view-only report card (student / parent)
    settings.tsx            user management (admin) + change own password
  components/
    app-shell.tsx           navbar, sidebar, breadcrumbs, Back, dark mode, role gate
    confirm-dialog.tsx      confirm-before-delete modal
    ui/                     shadcn/ui primitives
  lib/
    store.ts                data layer + models + grade/attendance helpers + CSV export
    auth.tsx                auth context: login, register, changePassword, logout
  styles.css                green + white design tokens (light + dark)
```

## Data models (`src/lib/store.ts`)

| Model | Fields |
| --- | --- |
| User | id, username, password, name, role (`admin` \| `teacher` \| `student` \| `parent`), assignedClassIds, studentId |
| Class | id, name, grade, section, teacherId |
| Student | id (auto), classId, fullName, rollNo, dob, parentsContact, feePaid |
| Subject | id, classId, name, totalMarks (default 100) |
| Marks | marks[studentId][subjectId] = number |
| Attendance | attendance[classId][date][studentId] = `P` \| `A` \| `L` |
| Notice | id, title, body, createdAt |

## Roles

- **Admin** — full access: classes, students, subjects, marks, attendance, reports, users.
- **Teacher** — only assigned classes; can enter marks/attendance; **cannot** add or delete classes.
- **Student** — view-only own report card (marks, grade, attendance) + notices.
- **Parent** — view-only linked child's data + notices.

Every role can change **their own** password from Settings. Admins link student/parent accounts to a student record in Settings.

## Adding a real backend later

Replace the `read`/`write` helpers in `src/lib/store.ts` (and the credential checks in `src/lib/auth.tsx`) with API calls or Lovable Cloud queries. Component code stays unchanged. Passwords in this demo are stored in plain text in the browser — hash them (bcrypt/Argon2) server-side as soon as a backend exists.
