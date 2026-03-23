# PSI Appraisal - Postman Testing Guide

Base URL: `http://localhost:8080`

---

## Recommended Testing Order

1. Create Department
2. Create HR user
3. Create Manager user
4. Create Employee user
5. Create Appraisal (HR)
6. Create Goals (Manager)
7. Submit Self Assessment (Employee)
8. Submit Manager Review (Manager)
9. Approve Appraisal (HR)
10. Acknowledge Appraisal (Employee)

---

## 1. Departments

### Create Department
```
POST /api/departments
Content-Type: application/json

{
  "name": "Engineering",
  "description": "Software engineering department"
}
```

### Get All Departments
```
GET /api/departments
```

### Get Department by ID
```
GET /api/departments/1
```

### Update Department
```
PUT /api/departments/1
Content-Type: application/json

{
  "name": "Engineering Updated",
  "description": "Updated description"
}
```

### Delete Department
```
DELETE /api/departments/1
```

---

## 2. Users

### Create HR User
> HR users cannot have a managerId — leave it out entirely.
```
POST /api/users
Content-Type: application/json

{
  "fullName": "Sarah HR",
  "email": "sarah.hr@company.com",
  "password": "password123",
  "role": "HR",
  "jobTitle": "HR Manager",
  "departmentId": 1
}
```

### Create Manager User
> Manager users — managerId is optional.
```
POST /api/users
Content-Type: application/json

{
  "fullName": "John Manager",
  "email": "john.manager@company.com",
  "password": "password123",
  "role": "MANAGER",
  "jobTitle": "Engineering Lead",
  "departmentId": 1
}
```

### Create Employee User
> Employees MUST have a managerId.
```
POST /api/users
Content-Type: application/json

{
  "fullName": "Ali Employee",
  "email": "ali.employee@company.com",
  "password": "password123",
  "role": "EMPLOYEE",
  "jobTitle": "Software Engineer",
  "departmentId": 1,
  "managerId": 2
}
```

### Get All Users
```
GET /api/users
```

### Get User by ID
```
GET /api/users/3
```

### Get My Profile
```
GET /api/users/me?userId=3
```

### Get Manager's Team
```
GET /api/users/manager/2/team
```

### Update User
```
PUT /api/users/3
Content-Type: application/json

{
  "fullName": "Ali Updated",
  "jobTitle": "Senior Software Engineer",
  "departmentId": 1,
  "managerId": 2
}
```

### Deactivate User (Soft Delete)
```
DELETE /api/users/3
```

---

## 3. Appraisals

### Create Appraisal (HR only)
> employeeId must be an EMPLOYEE, managerId must be a MANAGER.
```
POST /api/appraisals
Content-Type: application/json

{
  "cycleName": "Q1 2026",
  "cycleStartDate": "2026-01-01",
  "cycleEndDate": "2026-03-31",
  "employeeId": 3,
  "managerId": 2
}
```

### Get My Appraisals (Employee)
```
GET /api/appraisals/my?employeeId=3
```

### Get Team Appraisals (Manager)
```
GET /api/appraisals/team?managerId=2
```

### Get Appraisal by ID
```
GET /api/appraisals/1?requesterId=3
```

### Submit Self Assessment (Employee)
> Only works when appraisalStatus is PENDING.
```
PUT /api/appraisals/1/self-assessment?employeeId=3
Content-Type: application/json

{
  "whatWentWell": "I delivered the new payment module on time and mentored two junior developers.",
  "whatToImprove": "I need to improve my documentation habits and time estimation skills.",
  "achievements": "Completed the payment integration project, reduced API response time by 30%.",
  "selfRating": 4
}
```

### Submit Manager Review (Manager)
> Only works when appraisalStatus is SELF_SUBMITTED.
```
PUT /api/appraisals/1/manager-review?managerId=2
Content-Type: application/json

{
  "managerStrengths": "Ali consistently delivers quality work and is a reliable team member.",
  "managerImprovements": "Should focus on proactive communication and improving estimation accuracy.",
  "managerComments": "Overall a strong performer with good potential for growth.",
  "managerRating": 4
}
```

### Approve Appraisal (HR)
> Only works when appraisalStatus is MANAGER_REVIEWED.
```
PATCH /api/appraisals/1/approve
```

### Acknowledge Appraisal (Employee)
> Only works when appraisalStatus is APPROVED.
```
PATCH /api/appraisals/1/acknowledge?employeeId=3
```

---

## 4. Goals

### Create Goal (Manager)
> managerId must match the manager on the appraisal.
```
POST /api/goals?managerId=2
Content-Type: application/json

{
  "appraisalId": 1,
  "title": "Complete AWS Cloud Practitioner Certification",
  "description": "Enroll and complete the AWS Cloud Practitioner course by end of Q1.",
  "dueDate": "2026-03-31"
}
```

### Get Goal by ID
```
GET /api/goals/1
```

### Get Goals by Appraisal
```
GET /api/goals/appraisal/1
```

### Get Goals by Employee
```
GET /api/goals/employee/3
```

### Update Goal (Manager)
```
PUT /api/goals/1?managerId=2
Content-Type: application/json

{
  "title": "Complete AWS Solutions Architect Certification",
  "description": "Updated to Solutions Architect level.",
  "dueDate": "2026-03-31"
}
```

### Update Goal Progress (Employee)
```
PATCH /api/goals/1/progress?employeeId=3
Content-Type: application/json

{
  "progressPercent": 75,
  "status": "IN_PROGESS"
}
```

> Valid status values: `NOT_STARTED`, `IN_PROGESS`, `COMPLETED`, `CANCELLED`

### Delete Goal (Manager)
```
DELETE /api/goals/1?managerId=2
```

---

## 5. Feedback

### Submit Feedback
> feedbackType options: `SELF`, `PEER`, `MANAGER`
```
POST /api/feedback?reviewerId=2
Content-Type: application/json

{
  "appraisalId": 1,
  "revieweeId": 3,
  "comments": "Ali is a great team player and always willing to help others.",
  "rating": 4,
  "feedbackType": "MANAGER"
}
```

### Peer Feedback Example
```
POST /api/feedback?reviewerId=4
Content-Type: application/json

{
  "appraisalId": 1,
  "revieweeId": 3,
  "comments": "Great collaborator, always available for code reviews.",
  "rating": 5,
  "feedbackType": "PEER"
}
```

### Get Feedback for Appraisal
```
GET /api/feedback/appraisal/1
```

### Get Feedback Received by Employee
```
GET /api/feedback/employee/3
```

---

## 6. Notifications

### Get My Notifications
```
GET /api/notifications?userId=3
```

### Get Unread Count
```
GET /api/notifications/unread-count?userId=3
```

### Mark One Notification as Read
```
PATCH /api/notifications/1/read?userId=3
```

### Mark All Notifications as Read
```
PATCH /api/notifications/read-all?userId=3
```

---

## Appraisal Status Flow

```
PENDING
  → (employee submits self assessment)
SELF_SUBMITTED
  → (manager submits review)
MANAGER_REVIEWED
  → (HR approves)
APPROVED
  → (employee acknowledges)
ACKNOWLEDGED
```

---

## Role Rules

| Role     | managerId required? | Can be appraisal employee? | Can be appraisal manager? |
|----------|--------------------|-----------------------------|---------------------------|
| HR       | Not allowed        | No                          | No                        |
| MANAGER  | Optional           | No                          | Yes                       |
| EMPLOYEE | Required           | Yes                         | No                        |

---

## Common Errors

| Error | Reason |
|-------|--------|
| `Employees must be assigned a manager` | Creating EMPLOYEE without managerId |
| `HR users cannot have a manager assigned` | Passing managerId when creating HR |
| `Assigned manager must have the MANAGER role` | managerId points to non-MANAGER user |
| `Self-assessment already submitted` | Trying to submit self assessment twice |
| `Cannot review yet` | Manager reviewing before employee submits |
| `Cannot approve yet` | HR approving before manager reviews |
| `You have already submitted X feedback` | Duplicate feedback of same type |
