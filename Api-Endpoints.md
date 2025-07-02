# 📡 UniSkill+ API Endpoints

## ⚙️ Auth

| Method | Endpoint                    | Description                                   |
| -----: | --------------------------- | --------------------------------------------- |
|   POST | /api/auth/register-student  | Student signup (email, regNumber, dept, etc.) |
|   POST | /api/auth/register-lecturer | Lecturer signup (email, staffId, dept, etc.)  |
|   POST | /api/auth/login             | Login for all users                           |

---

## 🏛️ Universities

| Method | Endpoint              |                            Description |
| -----: | --------------------- | -------------------------------------: |
|   POST | /api/universities     | Create new university (by super admin) |
|    GET | /api/universities     |                  List all universities |
|    GET | /api/universities/:id |             Get single university info |

---

## 🏢 Departments

| Method |                             Endpoint |                    Description |
| -----: | -----------------------------------: | -----------------------------: |
|   POST | /api/universities/:uniId/departments |   Add department to university |
|    GET | /api/universities/:uniId/departments | List departments of university |
|    GET |                 /api/departments/:id |          Get single department |

---

## 📚 Courses

| Method |                         Endpoint |                Description |
| -----: | -------------------------------: | -------------------------: |
|   POST | /api/departments/:deptId/courses |   Add course to department |
|    GET | /api/departments/:deptId/courses | List courses in department |
|    GET |                 /api/courses/:id |          Get course detail |
|    PUT | /api/courses/:id/assign-lecturer |  Assign lecturer to course |

---

## 👩‍🏫 Lecturers

| Method |                           Endpoint |                  Description |
| -----: | ---------------------------------: | ---------------------------: |
|    GET | /api/universities/:uniId/lecturers | List lecturers in university |
|    GET |                 /api/lecturers/:id |         Get lecturer profile |

---

## 🎓 Students

| Method |                          Endpoint |                 Description |
| -----: | --------------------------------: | --------------------------: |
|    GET | /api/universities/:uniId/students | List students in university |
|    GET |                 /api/students/:id |         Get student profile |

---

## 🗂 Content

| Method |                       Endpoint |                               Description |
| -----: | -----------------------------: | ----------------------------------------: |
|   POST | /api/courses/:courseId/content | Upload course content (video, note, link) |
|    GET | /api/courses/:courseId/content |                       List course content |

---

## ❓ Quizzes

| Method |                       Endpoint |             Description |
| -----: | -----------------------------: | ----------------------: |
|   POST | /api/courses/:courseId/quizzes |     Add / generate quiz |
|    GET | /api/courses/:courseId/quizzes | List quizzes for course |
|   POST |    /api/quizzes/:quizId/submit | Student submits answers |

---

## 🏅 Certificates

| Method |                              Endpoint |                          Description |
| -----: | ------------------------------------: | -----------------------------------: |
|   POST |   /api/courses/:courseId/certificates | Issue certificate after passing quiz |
|    GET | /api/students/:studentId/certificates |          List student's certificates |

---

## 💬 Chat

| Method |                          Endpoint |              Description |
| -----: | --------------------------------: | -----------------------: |
|    GET | /api/channels/:channelId/messages | List messages in channel |
|   POST | /api/channels/:channelId/messages |         Post new message |
