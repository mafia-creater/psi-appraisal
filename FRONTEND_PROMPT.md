# Frontend Build Prompt — PSI Appraisal System

## Overview

Build a complete frontend for the PSI Appraisal System using **React**, **Tailwind CSS**, and **shadcn/ui**. The backend is a Spring Boot REST API running at `http://localhost:8080`.

This is an internal HR tool used by three types of users: **HR**, **Manager**, and **Employee**. Each role sees a completely different dashboard and set of features.

---

## Tech Stack

- React 18 + Vite
- TypeScript
- Tailwind CSS
- shadcn/ui (use Card, Button, Badge, Dialog, Form, Input, Textarea, Select, Table, Progress, Tabs, Avatar, DropdownMenu, Separator, Tooltip)
- React Router v6 (for routing)
- React Query (TanStack Query) for all API calls and caching
- React Hook Form + Zod for form validation
- Axios for HTTP requests
- date-fns for date formatting

---

## Authentication (Simulated)

There is no real auth/JWT in the backend yet. Simulate login by:

1. Show a login page with an email field and a "Select Role" dropdown (HR / MANAGER / EMPLOYEE).
2. On submit, call `GET /api/users` and find the user by email.
3. Store the matched user object (id, fullName, role, email, jobTitle, departmentName, managerId, managerName) in React Context and localStorage.
4. All subsequent API calls use the stored `userId` as a query param where required.
5. Show a "Logout" button that clears context and localStorage and redirects to login.

---

## Project Structure

```
src/
  api/           # axios instance + all API functions grouped by domain
  components/    # shared UI components (Navbar, Sidebar, StatusBadge, RatingStars, etc.)
  context/       # AuthContext
  hooks/         # custom React Query hooks
  pages/
    auth/        # LoginPage
    hr/          # HR-specific pages
    manager/     # Manager-specific pages
    employee/    # Employee-specific pages
    shared/      # Pages accessible by multiple roles
  types/         # TypeScript interfaces matching API responses
  lib/           # utils, cn helper
```

---

## API Base URL & Axios Setup

```ts
// src/api/axios.ts
const api = axios.create({ baseURL: 'http://localhost:8080' });
```

All responses follow this wrapper:
```ts
interface ApiResponse<T> {
  success: boolean;
  message: string;
  data: T;
}
```

---

## TypeScript Types

```ts
type Role = 'HR' | 'MANAGER' | 'EMPLOYEE';

type AppraisalStatus =
  | 'PENDING'
  | 'SELF_SUBMITTED'
  | 'MANAGER_REVIEWED'
  | 'APPROVED'
  | 'ACKNOWLEDGED';

type CycleStatus = 'DRAFT' | 'ACTIVE' | 'CLOSED';

type GoalStatus = 'NOT_STARTED' | 'IN_PROGESS' | 'COMPLETED' | 'CANCELLED';

type FeedbackType = 'SELF' | 'PEER' | 'MANAGER';

type NotificationType =
  | 'CYCLE_STARTED'
  | 'APPRAISAL_DUE'
  | 'SELF_ASSESSMENT_SUBMITTED'
  | 'MANAGER_REVIEW_DONE'
  | 'APPRAISAL_APPROVED'
  | 'FEEDBACK_RECEIVED'
  | 'GENERAL';

interface User {
  id: number;
  fullName: string;
  email: string;
  role: Role;
  jobTitle: string;
  departmentName: string | null;
  managerId: number | null;
  managerName: string | null;
  isActive: boolean;
  createdAt: string;
}

interface Appraisal {
  id: number;
  cycleName: string;
  cycleStartDate: string;
  cycleEndDate: string;
  cycleStatus: CycleStatus;
  employeeId: number;
  employeeName: string;
  employeeJobTitle: string;
  employeeDepartment: string | null;
  managerId: number;
  managerName: string;
  whatWentWell: string | null;
  whatToImprove: string | null;
  achievements: string | null;
  selfRating: number | null;
  managerStrengths: string | null;
  managerImprovements: string | null;
  managerComments: string | null;
  managerRating: number | null;
  appraisalStatus: AppraisalStatus;
  submittedAt: string | null;
  approvedAt: string | null;
  createdAt: string;
}

interface Goal {
  id: number;
  appraisalId: number;
  employeeId: number;
  employeeName: string;
  title: string;
  description: string;
  progressPercent: number;
  status: GoalStatus;
  dueDate: string;
}

interface Feedback {
  id: number;
  appraisalId: number;
  reviewerId: number;
  reviewerName: string;
  revieweeId: number;
  revieweeName: string;
  comments: string;
  rating: number;
  feedbackType: FeedbackType;
  createdAt: string;
}

interface Notification {
  id: number;
  title: string;
  message: string;
  type: NotificationType;
  isRead: boolean;
  createdAt: string;
}

interface Department {
  id: number;
  name: string;
  description: string;
}
```

---

## API Functions

### Users
```ts
GET    /api/users                          → User[]
GET    /api/users/:id                      → User
GET    /api/users/me?userId=               → User
GET    /api/users/manager/:managerId/team  → User[]
POST   /api/users                          → User
PUT    /api/users/:id                      → User
DELETE /api/users/:id                      → void
```

### Departments
```ts
GET    /api/departments      → Department[]
GET    /api/departments/:id  → Department
POST   /api/departments      → Department
PUT    /api/departments/:id  → Department
DELETE /api/departments/:id  → void
```

### Appraisals
```ts
POST   /api/appraisals                              → Appraisal   (HR)
GET    /api/appraisals/my?employeeId=               → Appraisal[] (Employee)
GET    /api/appraisals/team?managerId=              → Appraisal[] (Manager)
GET    /api/appraisals/:id?requesterId=             → Appraisal
PUT    /api/appraisals/:id/self-assessment?employeeId=  → Appraisal
PUT    /api/appraisals/:id/manager-review?managerId=    → Appraisal
PATCH  /api/appraisals/:id/approve                  → Appraisal   (HR)
PATCH  /api/appraisals/:id/acknowledge?employeeId=  → Appraisal
```

### Goals
```ts
POST   /api/goals?managerId=                  → Goal
GET    /api/goals/:id                         → Goal
GET    /api/goals/appraisal/:appraisalId      → Goal[]
GET    /api/goals/employee/:employeeId        → Goal[]
PUT    /api/goals/:id?managerId=              → Goal
PATCH  /api/goals/:id/progress?employeeId=   → Goal
DELETE /api/goals/:id?managerId=             → void
```

### Feedback
```ts
POST   /api/feedback?reviewerId=              → Feedback
GET    /api/feedback/appraisal/:appraisalId   → Feedback[]
GET    /api/feedback/employee/:employeeId     → Feedback[]
```

### Notifications
```ts
GET    /api/notifications?userId=             → Notification[]
GET    /api/notifications/unread-count?userId= → number
PATCH  /api/notifications/:id/read?userId=    → Notification
PATCH  /api/notifications/read-all?userId=    → void
```

---

## Pages & Features by Role

---

### Shared Layout

All authenticated pages share:
- A **sidebar** on the left with navigation links filtered by role
- A **top navbar** showing: app logo, user's name + role badge, notification bell with unread count badge, logout button
- Notification bell opens a dropdown panel showing the latest 10 notifications with mark-as-read and mark-all-read buttons
- Clicking a notification marks it as read

---

### Login Page (`/login`)

- Clean centered card with app logo/name "PSI Appraisal"
- Email input field
- Role selector (HR / MANAGER / EMPLOYEE) using shadcn Select
- "Sign In" button
- On submit: fetch all users, find by email, validate role matches, store in context
- Show error toast if email not found or role mismatch

---

### HR Dashboard (`/hr/dashboard`)

Summary cards showing:
- Total employees (active)
- Total appraisals this cycle
- Appraisals pending approval (status = MANAGER_REVIEWED)
- Appraisals completed (status = ACKNOWLEDGED)

Below the cards: a table of all appraisals across all cycles with columns:
`Employee | Department | Manager | Cycle | Status | Created | Actions`

Status shown as a colored Badge:
- PENDING → gray
- SELF_SUBMITTED → blue
- MANAGER_REVIEWED → yellow/orange
- APPROVED → green
- ACKNOWLEDGED → purple

Actions column: "View" button → opens appraisal detail page. "Approve" button visible only when status is MANAGER_REVIEWED.

---

### HR — Manage Users (`/hr/users`)

Full users table with columns:
`Name | Email | Role | Job Title | Department | Manager | Status | Actions`

- Role shown as Badge (HR=red, MANAGER=blue, EMPLOYEE=green)
- Active/Inactive shown as Badge
- "Add User" button opens a Dialog/Sheet with a form:
  - Full Name (required)
  - Email (required, email format)
  - Password (required)
  - Role selector (HR / MANAGER / EMPLOYEE)
  - Job Title
  - Department selector (fetched from /api/departments)
  - Manager selector — only shown when role is EMPLOYEE or MANAGER, populated with users who have role=MANAGER
- Edit button opens same form pre-filled
- Deactivate button with confirmation dialog

---

### HR — Manage Departments (`/hr/departments`)

Simple table: `Name | Description | Actions`

"Add Department" button opens a Dialog with name + description fields.
Edit and Delete actions per row.

---

### HR — Create Appraisal (`/hr/appraisals/create`)

Form with:
- Cycle Name (text input, e.g. "Q1 2026")
- Cycle Start Date (date picker)
- Cycle End Date (date picker)
- Employee selector (dropdown showing only EMPLOYEE role users with their job title and department)
- Manager selector (dropdown showing only MANAGER role users — auto-populated with the employee's assigned manager but editable)

On submit: POST /api/appraisals. Show success toast and redirect to appraisal detail.

---

### HR — Appraisal Detail (`/hr/appraisals/:id`)

Full read-only appraisal view showing:

**Header card:**
- Employee name, job title, department
- Manager name
- Cycle name, start/end dates
- Status badge + cycle status badge

**Self Assessment section** (shown after SELF_SUBMITTED):
- What Went Well
- What To Improve
- Achievements
- Self Rating (star display 1-5)

**Manager Review section** (shown after MANAGER_REVIEWED):
- Strengths
- Areas for Improvement
- Overall Comments
- Manager Rating (star display 1-5)

**Goals section:**
- List of goals with title, due date, progress bar, status badge

**Feedback section:**
- List of all feedback submitted for this appraisal (reviewer name, type badge, rating, comments)

**Action button:**
- "Approve" button shown only when status = MANAGER_REVIEWED → calls PATCH /api/appraisals/:id/approve

---

### Manager Dashboard (`/manager/dashboard`)

Summary cards:
- Team size (number of direct reports)
- Active appraisals (status != ACKNOWLEDGED)
- Awaiting my review (status = SELF_SUBMITTED)
- Completed (status = ACKNOWLEDGED)

Below: table of team appraisals from GET /api/appraisals/team?managerId=
Columns: `Employee | Cycle | Status | Self Rating | My Rating | Actions`

"Review" button on rows where status = SELF_SUBMITTED.

---

### Manager — Team (`/manager/team`)

Cards or table showing all direct reports from GET /api/users/manager/:managerId/team.
Each card shows: avatar (initials), name, job title, department, email.
Click on a team member → view their appraisals and goals.

---

### Manager — Appraisal Review (`/manager/appraisals/:id/review`)

Shows the full appraisal detail (same layout as HR detail view).

When status = SELF_SUBMITTED, show a form below the self-assessment section:
- Strengths (textarea, required)
- Areas for Improvement (textarea, required)
- Overall Comments (textarea, optional)
- Manager Rating (1-5 star selector, required)
- "Submit Review" button → PUT /api/appraisals/:id/manager-review?managerId=

---

### Manager — Goals (`/manager/goals`)

Table of all goals across the manager's team appraisals.
Columns: `Employee | Goal Title | Due Date | Progress | Status | Actions`

Progress shown as a shadcn Progress bar component.

"Add Goal" button opens a Dialog:
- Appraisal selector (dropdown of team appraisals)
- Title (required)
- Description (textarea)
- Due Date (date picker)

Edit and Delete actions per goal row.

---

### Employee Dashboard (`/employee/dashboard`)

Summary cards:
- My active appraisals
- Goals in progress
- Unread notifications

Below: list of my appraisals from GET /api/appraisals/my?employeeId=
Each appraisal shown as a card with: cycle name, dates, manager name, status badge, and a "View / Fill" button.

If status = PENDING → button says "Submit Self Assessment" (highlighted).
If status = APPROVED → button says "Acknowledge" (highlighted).
Otherwise → "View Details".

---

### Employee — Self Assessment Form (`/employee/appraisals/:id/self-assessment`)

Only accessible when appraisalStatus = PENDING.

Shows appraisal header info (cycle name, manager name, dates).

Form fields:
- What Went Well (textarea, required, placeholder: "Describe your key contributions and successes...")
- What Could I Improve (textarea, required, placeholder: "Be honest about areas where you could have done better...")
- Key Achievements (textarea, required, placeholder: "List specific achievements, metrics, projects completed...")
- Self Rating — interactive 1-5 star selector (required)

"Submit Self Assessment" button → PUT /api/appraisals/:id/self-assessment?employeeId=
On success: redirect to appraisal detail, show success toast.

---

### Employee — Appraisal Detail (`/employee/appraisals/:id`)

Same layout as HR detail view but from the employee's perspective.

Shows self-assessment (read-only after submission).
Shows manager review section only after MANAGER_REVIEWED.
Shows goals list with progress update capability.

"Acknowledge" button shown only when status = APPROVED → PATCH /api/appraisals/:id/acknowledge?employeeId=

---

### Employee — Goals (`/employee/goals`)

List of all my goals from GET /api/goals/employee/:employeeId.

Each goal shown as a card:
- Title + description
- Due date (color red if overdue)
- Progress bar (shadcn Progress)
- Status badge
- "Update Progress" button → opens a Dialog with:
  - Progress percent slider (0-100)
  - Status selector (NOT_STARTED / IN_PROGESS / COMPLETED / CANCELLED)

---

### Employee — Feedback (`/employee/feedback`)

Two tabs using shadcn Tabs:

**Tab 1 — Received Feedback:**
List of all feedback received from GET /api/feedback/employee/:employeeId.
Each item shows: reviewer name, feedback type badge, star rating, comments, date.

**Tab 2 — Give Feedback:**
Form to submit peer feedback:
- Appraisal selector (from my appraisals list)
- Reviewee (auto-filled as the employee in that appraisal — for peer feedback this would be a colleague)
- Comments (textarea, required)
- Rating (1-5 star selector, required)
- Feedback Type (PEER / SELF — MANAGER type is only for managers)
- Submit → POST /api/feedback?reviewerId=

---

## Shared Components to Build

### `StatusBadge`
Takes an `AppraisalStatus` and returns a colored shadcn Badge:
- PENDING → slate
- SELF_SUBMITTED → blue
- MANAGER_REVIEWED → amber
- APPROVED → green
- ACKNOWLEDGED → purple

### `RatingStars`
Props: `value: number`, `onChange?: (v: number) => void`, `readonly?: boolean`
Renders 5 stars, filled based on value. Interactive when onChange is provided.

### `GoalStatusBadge`
- NOT_STARTED → slate
- IN_PROGESS → blue
- COMPLETED → green
- CANCELLED → red

### `NotificationPanel`
Dropdown triggered by bell icon in navbar.
Shows list of notifications, unread ones highlighted.
"Mark all as read" button at top.
Each notification shows icon based on type, title, message, time ago.

### `AppraisalCard`
Reusable card showing appraisal summary: cycle name, employee/manager names, status badge, dates, action button.

### `UserAvatar`
Shows initials in a colored circle using shadcn Avatar. Color based on role.

---

## UX & Design Notes

- Use a clean, professional color scheme. Suggested: white background, slate sidebar, blue primary accent.
- All forms use React Hook Form + Zod validation. Show inline field errors.
- All API calls use React Query. Show loading skeletons (shadcn Skeleton) while fetching.
- Show toast notifications (shadcn Sonner or Toast) for all success and error actions.
- All destructive actions (delete, deactivate) require a shadcn AlertDialog confirmation.
- Tables should be responsive — on small screens collapse to cards.
- Empty states: when a list is empty, show a friendly illustration/message (e.g. "No appraisals yet").
- Dates formatted as "Jan 1, 2026" using date-fns format.
- The appraisal status flow is strictly enforced on the backend — the UI should hide/disable actions that aren't valid for the current status so users never hit a backend error.

---

## Routing Structure

```
/login

/hr/dashboard
/hr/users
/hr/departments
/hr/appraisals/create
/hr/appraisals/:id

/manager/dashboard
/manager/team
/manager/goals
/manager/appraisals/:id/review
/manager/appraisals/:id

/employee/dashboard
/employee/appraisals
/employee/appraisals/:id
/employee/appraisals/:id/self-assessment
/employee/goals
/employee/feedback
```

Route guards: if a user with role EMPLOYEE tries to access `/hr/*` or `/manager/*`, redirect to their dashboard. Same for other roles.

---

## Setup Commands

```bash
npm create vite@latest psi-appraisal-frontend -- --template react-ts
cd psi-appraisal-frontend
npm install
npx shadcn@latest init
npm install axios @tanstack/react-query react-router-dom react-hook-form zod @hookform/resolvers date-fns
npx shadcn@latest add button card badge dialog form input textarea select table tabs progress avatar dropdown-menu separator tooltip alert-dialog skeleton sonner
```
