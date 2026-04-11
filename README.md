# 🎓 Academic Coordination Management System

> A full-stack web application for academic coordinators to manage exams, grading deadlines, proficiency tests, thesis seminars, and team collaboration — with automated email generation and task tracking.

[![Demo](https://img.shields.io/badge/Live%20Demo-Firebase%20Hosted-blue?style=flat-square)](https://gerenciador-de-provas.firebaseapp.com)
[![Stack](https://img.shields.io/badge/Stack-Vanilla%20JS%20%2B%20Firebase-orange?style=flat-square)]()
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)]()

---

## 📌 Overview

Managing academic exams involves dozens of sequential deadlines: requesting exam drafts from professors, sending them to print, tracking corrections, verifying grade submissions, and communicating with students. Done manually, this is error-prone and time-consuming.

This system automates the entire workflow. Coordinators register a course and its exams; the app calculates every downstream deadline — adjusted for Brazilian national holidays and weekends — generates ready-to-send email templates, and surfaces a prioritized task board with real-time status tracking.

---

## ✨ Key Features

### 📋 Automated Task Board
- Deadline engine that computes all workflow steps from a single course start date
- Status classification: **Overdue**, **Attention** (due today), **In Progress**, **Completed**
- Chip-based filters by status and item type (course, proficiency test, dependency exam, seminar)
- Progress bar and step counter per item

### 📧 One-Click Email Generation
- Context-aware email templates for every workflow step:
  - Syllabi request, exam draft request, verification follow-ups, correction deadlines, grade submission reminders
  - Library print requests with copy count
  - Student communications for proficiency tests and seminars
- Formatted with bold highlights; copies with HTML formatting preserved for Gmail/Outlook paste
- `mailto:` integration with pre-filled subject and body

### 🗓️ Smart Deadline Calculation
- All deadlines computed using a Brazilian holiday calendar (fixed + Easter-relative moveable feasts)
- Deadlines that fall on weekends or holidays are automatically shifted to the nearest valid business day

### 👥 Team Collaboration (Work Groups)
- Admins create named work groups and assign coordinators
- Items (courses, tests, seminars) can be shared with one or more groups
- Each coordinator sees a unified task board filtered by their active group context
- Ownership model: only the creator or an admin can edit/delete items

### 🔐 Authentication & Role Management
- Firebase Authentication with email/password
- Admin and Coordinator roles; first-run setup flow creates the initial admin account
- Force-password-change on first login; self-service password reset via email link
- Admin panel for user creation, deactivation, and role changes

### 📦 Module Coverage
| Module | Description |
|---|---|
| **Disciplinas** (Courses) | Register courses with multiple exams and assignments; full CRUD |
| **Proficiência** | Language proficiency test scheduling; student communication workflow |
| **Dependência** | Makeup exam management for multiple subjects in a single sitting |
| **Seminários** | Thesis seminar lifecycle: abstract collection → calendar event (.ics) → article distribution |
| **Tarefas** | Unified, prioritized task board across all modules |
| **Admin** | User & group management, credential distribution |

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML5, CSS3, JavaScript (ES2020+) |
| Backend / DB | Firebase Firestore (NoSQL, real-time) |
| Auth | Firebase Authentication |
| Hosting | Firebase Hosting |
| Architecture | Single-page application (SPA), single-file deployment |

**No build step. No framework dependency. No npm.** The entire application ships as one self-contained `index.html` file — intentionally chosen for zero-friction deployment in institutional environments where toolchains cannot be assumed.

---

## 🏗️ Architecture Highlights

### Deadline Engine
```
registerCourse(startDate, exams[]) 
  → generateEvents(course)          # computes all workflow steps
    → proximoDiaUtil(date)          # adjusts to next valid business day
    → getFeriados(year)             # Easter algorithm + fixed holidays
  → calcSituacao(pendingEvents)     # classifies: overdue / attention / in-progress / done
```

### Data Model (Firestore)
```
users/{uid}                         # profile, role, mustChangePassword
userData/{uid}                      # disciplinas[], proficiencias[], dependencias[], 
                                    # seminarios[], emailStatus{}
groups/{groupId}                    # nome, members[]
_config/setup                       # initialization flag
```

### Multi-User Aggregation
Items are stored per-owner in `userData` but rendered aggregated by group context. The `_getAggregated(field)` function merges data from all relevant owners filtered by `itemMatchesContext()`, so coordinators in the same group share a single unified view without duplicating data.

---

## 🚀 Getting Started

### Prerequisites
- A [Firebase project](https://console.firebase.google.com/) with Firestore and Authentication enabled

### Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/nadiacmoreira2003/gerenciador-de-tarefas-academicas.git
   cd gerenciador-de-tarefas-academicas
   ```

2. Replace the Firebase configuration in `index.html`:
   ```javascript
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT.firebaseapp.com",
     projectId: "YOUR_PROJECT_ID",
     storageBucket: "YOUR_PROJECT.appspot.com",
     messagingSenderId: "YOUR_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   ```

3. Open `index.html` in a browser — the app will detect no prior setup and prompt you to create the first admin account.

4. *(Optional)* Deploy to Firebase Hosting:
   ```bash
   npm install -g firebase-tools
   firebase login
   firebase init hosting
   firebase deploy
   ```

### Firestore Security Rules (recommended)
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid} {
      allow read: if request.auth != null;
      allow write: if request.auth.uid == uid 
                   || get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    match /userData/{uid} {
      allow read, write: if request.auth != null;
    }
    match /groups/{groupId} {
      allow read: if request.auth != null;
      allow write: if get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    match /_config/{doc} {
      allow read, write: if request.auth != null;
    }
  }
}
```

---

## 📸 Screenshots

### Task Board — Real-Time Deadline Tracking (Tarefas)
<img width="1141" height="815" alt="Captura de Tela 2026-04-11 às 7 00 43 PM" src="https://github.com/user-attachments/assets/556e7a27-5121-4f35-b756-44925e367d3f" />

### Automated Email Generation
<img width="529" height="394" alt="Captura de Tela 2026-04-11 às 7 05 45 PM" src="https://github.com/user-attachments/assets/cbdb5df8-550b-4a1c-94a6-ae74feaebbd5" />

### Course Registration with Exam Scheduling
<img width="803" height="638" alt="Captura de Tela 2026-04-11 às 7 06 39 PM" src="https://github.com/user-attachments/assets/39c9ebb4-4501-4a27-95e8-b2d48dd6b4d6" />

### Admin Panel — Users & Work Groups
<img width="727" height="784" alt="Captura de Tela 2026-04-11 às 7 07 14 PM" src="https://github.com/user-attachments/assets/305b486d-14d5-46be-be73-6f9dd9ef77b4" />

---

## 🧠 Design Decisions & Lessons Learned

- **Single-file SPA**: Eliminates deployment friction for non-technical institutional staff. The tradeoff is a larger initial payload, acceptable for an internal tool with a small user base.
- **Client-side deadline engine**: All date math runs in the browser. This makes the app fully functional offline after first load and avoids serverless function costs.
- **Firestore over SQL**: The schema-free model allowed rapid iteration as new workflow modules (seminars, dependency exams) were added without migrations.
- **Holiday-aware scheduling**: The Easter algorithm (Anonymous Gregorian) is computed at runtime for any year, so the system remains accurate indefinitely without a hardcoded holiday table.

---

## 👩‍💻 Author

**Nadia Cardoso Moreira**  
Associate Professor & Graduate Program Coordinator — FUCAPE Business School, Brazil  
PhD in Accounting & Management | MSc & BSc in Mathematics  
Research: Causal Inference · NLP for Finance · Quantitative Methods

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/nadia-cardoso-moreira-302b4220/?locale=pt)
---

## 📄 License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.
