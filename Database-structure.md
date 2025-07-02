# 🛠️ **UniSkill+ Backend – How universities, lecturers & students are onboarded and linked**

This explains:
✅ Registration flow for universities, lecturers & students  
✅ How they’re connected in the database  
✅ Why this design is clean, scalable, and secure

---

## 🏛️ Step 1: Universities register & onboard

- **Platform Admin** creates an account for each university (or universities apply & get approved)
- When approved, the university gets:
  - A unique `universityId`
  - Admin dashboard to manage departments, lecturers, and courses

**DB: Universities**
| Field | Type |
|----------------|--------:|
| \_id | ObjectId|
| name | String |
| domain/emailSuffix | String (e.g., @unilag.edu.ng) |
| approved | Boolean |
| createdAt | Date |

---

## 🏢 Step 2: University creates departments & courses

Inside their dashboard, the university admin:

- Creates departments (e.g., Computer Science)
- Adds official courses (e.g., CSC101, ACC201)

**DB: Departments**
| Field | Type |
|--------------|---------:|
| \_id | ObjectId |
| universityId | ObjectId |
| name | String |

**DB: Courses**
| Field | Type |
|--------------|---------:|
| \_id | ObjectId |
| departmentId | ObjectId |
| lecturerId | ObjectId (can be null at first) |
| name | String |
| description | String |

---

## 👩‍🏫 Step 3: Lecturers register & get verified

- Lecturer signs up with:
  - Email (preferably matching university domain)
  - Name, phone
  - Staff/lecturer ID for verification
- Backend verifies:
  - Does lecturer belong to the registered university (via email domain or manual approval)?
- Lecturer is linked to:
  - `universityId`
  - One or more `departmentId`
  - One or more `courseId` they teach

**DB: Users**  
Lecturers and students can live in the same `users` table, differentiated by `role`.

| Field        |                                       Type |
| ------------ | -----------------------------------------: |
| \_id         |                                   ObjectId |
| name         |                                     String |
| email        |                                     String |
| passwordHash |                                     String |
| role         | Enum: 'lecturer' / 'student' / 'uni-admin' |
| universityId |                                   ObjectId |
| departmentId |                                   ObjectId |
| staffId      |                    String (only lecturers) |

---

## 🎓 Step 4: Students register & get verified

- Student signs up with:
  - Email (can be any, but optionally must match university domain)
  - Registration number / matric number
  - Select department
- Backend checks:
  - Does `regNumber` match the university records? (integration or manual upload)
- Student is linked to:
  - `universityId`
  - `departmentId`

---

## 🧩 Step 5: Linking everything together

|     Entity |                                                                  Links to |
| ---------: | ------------------------------------------------------------------------: |
| University |                                 Has many departments, lecturers, students |
| Department |          Belongs to one university; has many courses, students, lecturers |
|   Lecturer | Belongs to one university & one or more departments; teaches many courses |
|    Student |           Belongs to one university & department; enrolls in many courses |
|     Course |    Belongs to one department; taught by a lecturer; has content & quizzes |

---

## 📦 Example Database Collections (high-level)

```plaintext
Universities
├── _id, name, emailSuffix

Departments
├── _id, universityId, name

Users
├── _id, name, email, passwordHash, role: student / lecturer / admin
├── universityId, departmentId, regNumber / staffId

Courses
├── _id, name, departmentId, lecturerId

Content
├── _id, courseId, type (video, note, link), storageUrl

Quizzes
├── _id, courseId, questions (AI generated)

Certificates
├── _id, studentId, courseId, blockchainTxHash
```
