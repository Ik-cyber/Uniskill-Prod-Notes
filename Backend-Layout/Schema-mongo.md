```js
// 📦 models/university.model.js
const mongoose = require("mongoose");
const { Schema } = mongoose;

const UniversitySchema = new Schema(
  {
    name: { type: String, required: true },
    emailSuffix: String, // e.g., "@unilag.edu.ng"
    approved: { type: Boolean, default: false },
  },
  { timestamps: true },
);

module.exports = mongoose.model("University", UniversitySchema);

// 🏢 models/department.model.js
const DepartmentSchema = new Schema(
  {
    name: { type: String, required: true },
    universityId: {
      type: Schema.Types.ObjectId,
      ref: "University",
      required: true,
    },
  },
  { timestamps: true },
);

module.exports = mongoose.model("Department", DepartmentSchema);

// 👤 models/user.model.js
const UserSchema = new Schema(
  {
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    passwordHash: { type: String, required: true },
    role: {
      type: String,
      enum: ["student", "lecturer", "uni-admin"],
      required: true,
    },
    universityId: {
      type: Schema.Types.ObjectId,
      ref: "University",
      required: true,
    },
    departmentId: { type: Schema.Types.ObjectId, ref: "Department" },
    regNumber: String, // only for students
    staffId: String, // only for lecturers
  },
  { timestamps: true },
);

module.exports = mongoose.model("User", UserSchema);

// 📚 models/course.model.js
const CourseSchema = new Schema(
  {
    name: { type: String, required: true },
    description: String,
    departmentId: {
      type: Schema.Types.ObjectId,
      ref: "Department",
      required: true,
    },
    lecturerId: { type: Schema.Types.ObjectId, ref: "User" }, // optional at first
  },
  { timestamps: true },
);

module.exports = mongoose.model("Course", CourseSchema);

// 🗂 models/content.model.js
const ContentSchema = new Schema(
  {
    courseId: { type: Schema.Types.ObjectId, ref: "Course", required: true },
    type: { type: String, enum: ["video", "note", "link"], required: true },
    storageUrl: String, // for files / video
    text: String, // for notes
  },
  { timestamps: true },
);

module.exports = mongoose.model("Content", ContentSchema);

// ❓ models/quiz.model.js
const QuizSchema = new Schema(
  {
    courseId: { type: Schema.Types.ObjectId, ref: "Course", required: true },
    questions: [
      {
        question: String,
        options: [String],
        correctAnswer: String,
      },
    ],
  },
  { timestamps: true },
);

module.exports = mongoose.model("Quiz", QuizSchema);

// 🏅 models/certificate.model.js
const CertificateSchema = new Schema(
  {
    studentId: { type: Schema.Types.ObjectId, ref: "User", required: true },
    courseId: { type: Schema.Types.ObjectId, ref: "Course", required: true },
    blockchainTxHash: String,
  },
  { timestamps: true },
);

module.exports = mongoose.model("Certificate", CertificateSchema);

// 💬 models/chatMessage.model.js (optional)
const ChatMessageSchema = new Schema({
  channelId: String,
  userId: { type: Schema.Types.ObjectId, ref: "User", required: true },
  message: String,
  createdAt: { type: Date, default: Date.now },
});

module.exports = mongoose.model("ChatMessage", ChatMessageSchema);
```

## Structure

```text
/models
├─ university.model.js
├─ department.model.js
├─ user.model.js
├─ course.model.js
├─ content.model.js
├─ quiz.model.js
├─ certificate.model.js
└─ chatMessage.model.js
```

# 📝 Enrollments for UniSkill+

---

## 🔄 Enrollment Flow

1. Student registers & selects department
2. System auto-recommends or auto-enrolls student in required courses of that department
3. Student can view their enrolled courses (from Enrollment collection)
4. As student progresses, status updates:
   - `enrolled` → `in-progress` → `completed`
5. When completed, backend can trigger certificate issuance

---

## 🛠️ API Routes for Enrollment Management

| Method | Endpoint                             | Description                       |
| ------ | ------------------------------------ | --------------------------------- |
| GET    | /api/students/:studentId/enrollments | Get all enrollments for a student |
| POST   | /api/students/:studentId/enrollments | Enroll a student in a course      |
| PUT    | /api/enrollments/:id                 | Update enrollment status          |
| DELETE | /api/enrollments/:id                 | Withdraw/drop from a course       |

---

## 💡 Notes

- Enrollment ensures student-course relationships are flexible and trackable
- Supports future features like progress tracking, analytics, and notifications
- Status values help track student journey per course
- You can extend with grades, quiz results linked here if desired

## 📄 Enrollment Schema (MongoDB / Mongoose)

```js
const mongoose = require("mongoose");
const { Schema } = mongoose;

const EnrollmentSchema = new Schema(
  {
    studentId: { type: Schema.Types.ObjectId, ref: "User", required: true },
    courseId: { type: Schema.Types.ObjectId, ref: "Course", required: true },
    status: {
      type: String,
      enum: ["enrolled", "in-progress", "completed", "dropped"],
      default: "enrolled",
    },
    enrolledAt: { type: Date, default: Date.now },
    completedAt: Date,
  },
  { timestamps: true },
);

module.exports = mongoose.model("Enrollment", EnrollmentSchema);
```
