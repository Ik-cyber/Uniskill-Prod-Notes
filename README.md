# 🚀 UniSkill+ Hackathon Deep Technical Plan

> ✅ Full step-by-step breakdown: what to build, in what order, and how — for backend, frontend, mobile & AI.

---

## ✅ 0. Initial planning

- Define MVP: lecturer uploads → AI generates quiz → student views & completes quiz → badge issued → AI recommends next skill.
- Tech stack:
  - Backend: Node.js + Express / NestJS
  - Frontend: Next.js
  - Mobile: React Native
  - AI: OpenAI API
  - Blockchain: Solidity + Polygon testnet (Mumbai)
  - DB: PostgreSQL

### Project structure

```
/backend
/frontend
/mobile
/docs
```

---

## 🧰 1. Backend tasks (Backend Dev)

### 📦 Setup

```bash
npx create-nest-app backend
npm install axios openai pg typeorm cors dotenv
```

`.env`:

```
OPENAI_API_KEY=...
DB_URL=...
POLYGON_RPC_URL=...
```

### 📊 DB Models (TypeORM / Prisma)

| Table   | Fields                                             |
| ------- | -------------------------------------------------- |
| Users   | id, name, email, role (lecturer/student)           |
| Courses | id, title, description, video_url, lecturer_id     |
| Quizzes | id, course_id, question, options[], correct_answer |
| Badges  | id, student_id, course_id, tx_hash                 |

### 🔧 API Endpoints

| Method | Endpoint                         | Purpose                                    |
| ------ | -------------------------------- | ------------------------------------------ |
| POST   | /api/upload-course               | Lecturer uploads video & metadata          |
| POST   | /api/generate-quiz               | Call OpenAI to create quiz from transcript |
| GET    | /api/student/courses             | List available courses                     |
| GET    | /api/student/:id/quiz            | Get quiz questions                         |
| POST   | /api/student/:id/submit-quiz     | Check answers & issue badge                |
| GET    | /api/student/:id/badges          | Show earned badges                         |
| GET    | /api/student/:id/recommendations | AI skill recommendations                   |

### 🧠 AI integration

- Service file: `/backend/src/services/aiService.ts`
- Use OpenAI SDK:

```javascript
const openai = new OpenAI(process.env.OPENAI_API_KEY);
const completion = await openai.chat.completions.create({ ... });
```

- Store quiz & notes in DB.

### 🔗 Blockchain badge issuing

- Solidity smart contract:

```solidity
contract BadgeIssuer {
    mapping(address => string[]) public badges;
    function issueBadge(address student, string memory badgeId) public { ... }
}
```

- Deploy on Polygon Mumbai using Hardhat.
- Backend calls:

```javascript
const tx = await contract.issueBadge(studentAddress, badgeId);
```

---

## 🖥 2. Frontend tasks (Frontend Dev)

### ⚙️ Setup

```bash
npx create-next-app frontend
npm install axios ethers tailwindcss
```

### 📂 Pages & components

```
/pages
  /lecturer/upload.tsx
  /student/dashboard.tsx
  /student/quiz.tsx
/components
  Navbar, CourseCard, BadgeCard, QuizForm
```

### 🔧 Features

- **Upload page:**
  - Form: title, description, upload
  - Calls `/api/upload-course` and `/api/generate-quiz`
- **Student dashboard:**
  - Fetch & show notes, quiz, badges, AI recommendations
- **Quiz page:**
  - Show quiz, submit answers, get badge link

### 🧪 Blockchain

```javascript
import { ethers } from "ethers";
const provider = new ethers.JsonRpcProvider(POLYGON_RPC_URL);
```

---

## 📱 3. Mobile tasks (Mobile Dev)

### ⚙️ Setup

```bash
npx create-expo-app mobile
npm install axios ethers react-navigation
```

### 📂 Screens

- **DashboardScreen**: recommendations & badges
- **QuizScreen**: display quiz & submit
- **BadgeScreen**: show badge & tx link

---

## 🎨 4. UI/UX designer tasks

- Design wireframes: upload page, dashboard, quiz, badge
- Pick colors, fonts, export icons
- Share assets with frontend & mobile devs

---

## 🔗 5. Integration & flow

1. Lecturer uploads course → backend saves
2. Backend triggers AI → stores quiz & notes
3. Student dashboard fetches & shows data
4. Student submits quiz → backend issues badge
5. Student sees badge (block explorer link)
6. AI recommends next skill

---

## 📜 6. AI Prompts

- **Notes:** "Summarize this transcript into bullet notes for students."
- **Quiz:** "Generate 5 MCQs from this text with answers & wrong options."
- **Recommendation:** "Student completed digital skills course. Suggest 3 future-ready skills."

---

## 🧪 7. Testing & demo prep

- Seed DB with 1–2 sample courses
- Test full flow
- Make slides & short pitch script

---

## ✅ 8. Deliverables

- MVP live demo (Polygon testnet)
- Code on GitHub
- PDF pitch
- (Optional) demo video

---

## 🧩 Why this order?

- Backend & AI first → unblock frontend & mobile
- UI/UX design feeds early dev
- Blockchain connects once flow tested
- Mobile dev can reuse backend APIs

---

✨ **Done!**

Use this as your dev plan, README, or team doc 🚀
