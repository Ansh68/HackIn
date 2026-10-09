# 🚀 HackIn — The Ultimate Hackathon Platform

> **HackIn** is a full-stack MERN web platform that connects hackathon participants, organizers, and sponsors — all in one place. Find teammates, organize events, showcase projects, and build your developer identity.

---

## 📌 Table of Contents

1. [Problem Statement](#-problem-statement)
2. [What HackIn Solves](#-what-hackin-solves)
3. [Tech Stack — Why MERN?](#-tech-stack--why-mern)
4. [Database Design](#-database-design)
5. [Why MongoDB over SQL?](#-why-mongodb-over-sql)
6. [GitHub OAuth — How It Works](#-github-oauth--how-it-works)
7. [Why OAuth Instead of Passwords?](#-why-oauth-instead-of-passwords)
8. [Team Features — Deep Dive](#-team-features--deep-dive)
9. [Email Notifications — How They Work](#-email-notifications--how-they-work)
10. [Real-Time Chat — Socket.io](#-real-time-chat--socketio)
11. [Dev Feed — Social Layer](#-dev-feed--social-layer)
12. [Leaderboard & Ranking System](#-leaderboard--ranking-system)
13. [Sponsor Module](#-sponsor-module)
14. [Hackathon Management](#-hackathon-management)
15. [Media Uploads — Cloudinary](#-media-uploads--cloudinary)
16. [Project Showcase](#-project-showcase)
17. [API Architecture](#-api-architecture)
18. [Folder Structure](#-folder-structure)
19. [Environment Variables](#-environment-variables)
20. [Local Setup](#-local-setup)
21. [Interview Q&A — Important Questions](#-interview-qa--important-questions)

---

## 🧩 Problem Statement

The hackathon ecosystem is **fragmented**. Participants face these problems every time:

- 🔍 **No central platform** to discover upcoming hackathons
- 👥 **No organized way to find teammates** with matching skills
- 📂 **No structured place to showcase** hackathon projects
- 📣 **Organizers struggle** to reach the right audience and manage applications
- 💰 **Sponsors have no visibility** into which hackathons to fund
- 💬 **Teams communicate on random apps** with no context or persistence
- 🏆 **No gamification or reputation system** to reward consistent hackers

---

## ✅ What HackIn Solves

| Problem | HackIn Feature |
|---------|---------------|
| Can't find teammates | HackMates — Browse devs by skill, experience level |
| No hackathon discovery | Hackathon listing with filters (Online/Offline/Hybrid, Track, Prizes) |
| No team management | Create teams, send join requests, accept/reject members |
| No showcase | Project portfolio — add GitHub links, live demos, tech stacks |
| No real-time team chat | Socket.io powered team chatroom |
| Sponsor connect | Dedicated sponsor module with request workflow |
| No reputation system | Contribution score → Rank progression (Newbie → Legend) |
| No social layer | Dev Feed — posts with code snippets, images, likes & comments |
| No notifications | Email alerts for join requests, accept/reject events |

---

## 🛠 Tech Stack — Why MERN?

### **M — MongoDB**
- Schema-flexible document model fits hackathon data perfectly (no two hackathons are identical)
- JSON-like BSON documents map naturally to JavaScript objects
- Rich querying with `populate()` for relational-like joins
- Horizontal scaling for growing user bases

### **E — Express.js**
- Minimal, unopinionated Node.js framework
- Clean middleware pipeline for CORS, sessions, file uploads
- Easy to structure with RESTful routes and controllers

### **R — React.js (with Vite)**
- Component-based UI allows rapid iteration
- React Router v7 for SPA navigation
- Socket.io-client for real-time features
- React Quill for rich text editor in posts
- Lucide-React for consistent iconography

### **N — Node.js**
- Unified JavaScript across frontend and backend → no context switching
- Event-driven, non-blocking I/O ideal for real-time features
- Huge ecosystem (npm)

### **Additional Technologies**
| Technology | Purpose |
|-----------|---------|
| Firebase Auth + Admin SDK | GitHub OAuth token generation & verification |
| Socket.io | Real-time bidirectional team chat |
| Nodemailer (Gmail SMTP) | Transactional email notifications |
| Cloudinary | Media storage for post images and sponsor logos |
| Multer | File upload middleware (local buffer before Cloudinary) |
| connect-mongo + express-session | Server-side session management |
| Tailwind CSS v4 | Utility-first UI styling |

---

## 🗄 Database Design

The database (`hackin`) is organized around **5 core collections** with cross-collection references using MongoDB ObjectIds.

### 📊 Entity Relationship Overview

```
User ──────┬────── teams[] ──────── Team
           │                         │
           │                         ├── teamLeader (User ref)
           │                         ├── teamMembers[] (User ref)
           │                         └── joinRequests[] (User ref)
           │
           └── myRequests[]          Team ──── Hackathon
                                               │
Feed ────── userId (User ref)                  ├── participants[] (Team ref)
           ├── likes[] (User ref)              └── applications[] (Team ref)
           └── comments[].userId (User ref)
                                    Project ── userId (User ref)
Message ── teamId (Team ref)
           └── senderId (User ref)
```

### 📋 Schema Details

#### `User` Collection
```js
{
  oauthProvider: "github" | "email",  // Auth provider
  oauthId: String,                     // Firebase UID (unique)
  name, email, username,               // Identity
  profileImage, bio,                   // Profile
  skills: [String],                    // e.g. ["React", "Node"]
  socialLinks: { linkedin, portfolio },
  experienceLevel: "Beginner" | "Intermediate" | "Advanced",
  pastHackathons: [{ name, year, position }],
  contributionScore: Number,           // Gamification points
  rankingLevel: "Newbie" | "Rookie" | "Pro" | "Elite" | "Legend",
  teams: [ObjectId → Team],            // Teams joined
  myRequests: [{ teamId, status }],    // Join request history
  firstLogin: Boolean                  // Profile completion gate
}
```

#### `Hackathon` Collection
```js
{
  name, organizer (→ User), description,
  startDate, endDate, registrationDeadline,
  location: { address, city, state, country, postalCode },
  mode: "Online" | "Offline" | "Hybrid",
  track: "AI" | "Web3" | "AR/VR" | "IOT" | "Cyber Security" | ...,
  prizePool, prizes: { first, second, third },
  minTeamSize, maxTeamSize,
  participants: [ObjectId → Team],     // Accepted teams
  applications: [ObjectId → Team],     // Pending applications
  winner: { first, second, third } (→ Team),
  sponsors: [{ name, logo }],
  colorTheme, website, collegeRepresenting
}
```

#### `Team` Collection
```js
{
  teamName, description, hackathonName,
  teamSize, teamLeader (→ User),
  teamMembers: [ObjectId → User],
  skills: [String],                    // Aggregate of member skills
  teamCode: String (6-char crypto hex), // Unique join code
  lookingFor: [String],               // "Frontend Dev", "AI Engineer"
  joinRequests: [{ userId, message }],
  teamScore: Number,                   // Auto-calculated from member scores
  isTeamFull: Boolean,                 // Auto-flag via pre-save hook
  isLive: Boolean,                     // Active/Inactive
  dates: { startDate, endDate },
  location: String
}
```

#### `Feed` Collection
```js
{
  userId (→ User),
  content: String,                     // Post text
  image, video, codeSnippet, githubLink,
  likes: [ObjectId → User],           // Toggle array
  comments: [{ userId, text, createdAt }]
}
```

#### `Project` Collection
```js
{
  userId (→ User),
  hackathonName, projectTitle, description,
  teamName, achievement: "Winner" | "Runner-up" | "Participant",
  techStack: [String],
  githubLink, liveDemo,
  images: [String]                     // Cloudinary URLs
}
```

#### `Message` Collection (for Socket.io persistence)
```js
{
  teamId (→ Team),
  senderId (→ User),
  senderName: String,
  content: String,
  type: "message"
}
```

### 🔗 Key Design Decisions

1. **Embedded vs Referenced**: `pastHackathons` is embedded in User (rarely changes, always accessed together). `teamMembers` is referenced (queried independently for leaderboard).
2. **Pre-save Middleware**: The Team schema uses a `pre('save')` hook to auto-calculate `teamScore` and `isTeamFull` — no manual sync needed.
3. **Session Storage in MongoDB**: Using `connect-mongo`, sessions are stored in the same DB, so no external Redis needed.

---

## ❓ Why MongoDB over SQL?

| Criteria | MongoDB (Chosen) | SQL (PostgreSQL/MySQL) |
|---------|-----------------|------------------------|
| Schema Flexibility | ✅ Hackathon tracks, prizes, locations vary per event | ❌ Requires migrations for every schema change |
| Developer Speed | ✅ JS objects map directly to documents | ❌ ORM boilerplate or raw SQL |
| Nested Data | ✅ `comments[]` inside Feed document | ❌ Separate table + JOIN every time |
| Real-time scaling | ✅ Horizontal sharding built-in | ❌ Vertical scaling is expensive |
| Prototype speed | ✅ Schema changes without downtime | ❌ ALTER TABLE migrations |
| Mongoose ODM | ✅ JS-native, rich validation | ❌ Requires separate ORM (Sequelize, TypeORM) |

**Specific use case justification:**
- A hackathon can have `0` or `N` sponsors, `0-3` winners, optional location — MongoDB handles this without `NULL`-heavy columns
- The `Feed` post with embedded `comments[]` and `likes[]` avoids expensive JOIN queries on every feed load
- User profiles have optional fields (bio, portfolio, pastHackathons) that MongoDB handles natively

---

## 🔐 GitHub OAuth — How It Works

HackIn uses **Firebase Authentication** as the OAuth bridge to GitHub. Here's the exact flow:

### Step-by-Step Flow

```
User Clicks "Login with GitHub"
         │
         ▼
[Frontend] firebase.js → signInWithPopup(auth, githubProvider)
         │
         ▼
[GitHub] OAuth consent screen
  "HackIn wants access to your: email, username, profile"
         │
         ▼
[Firebase] Receives GitHub OAuth token
  → Mints a Firebase ID Token (JWT)
  → Also returns oauthAccessToken (GitHub's own token)
         │
         ▼
[Frontend] Uses GitHub token to call → GET https://api.github.com/user
  → Extracts: name, login (GitHub username)
         │
         ▼
[Frontend] Sends to Backend:
  POST /api/v1/auth/github
  Body: { token (Firebase JWT), name, username }
         │
         ▼
[Backend] auth.routes.js
  → admin.auth().verifyIdToken(token)  ← Firebase Admin SDK
  → Decodes: { uid, email, picture }
         │
         ▼
[Backend] MongoDB check:
  User.findOne({ oauthId: decodedToken.uid })
  ├── Found → Return existing user
  └── Not Found → Create new user document
         │
         ▼
[Backend] Returns user object to frontend
[Frontend] Stores user in localStorage/state
```

### Key Code Files
- **`frontend/src/firebase.js`** — Firebase initialization, `signInWithPopup`, token extraction
- **`backend/src/routes/auth.routes.js`** — Firebase Admin SDK token verification, user upsert

### Why Firebase as the Middle Layer?
Firebase abstracts the OAuth complexity. Without Firebase, you'd need to:
1. Register a GitHub OAuth App
2. Handle the redirect callback yourself
3. Exchange `code` for `access_token` manually
4. Verify tokens manually

Firebase handles all of this and provides a **signed JWT** that the backend can verify cryptographically.

---

## 🛡 Why OAuth Instead of Passwords?

| Concern | Password Auth | GitHub OAuth |
|---------|--------------|-------------|
| Security | ❌ Passwords stored (even hashed, breach risk) | ✅ No password stored — zero breach risk |
| Password reuse attacks | ❌ Users reuse passwords across sites | ✅ Not applicable |
| Developer audience | ❌ Extra friction — another form | ✅ Every developer has GitHub — 1 click login |
| Trust | ❌ Users must trust YOUR site | ✅ GitHub is already trusted |
| Profile data | ❌ Users fill forms manually | ✅ Name, email, avatar auto-fetched from GitHub |
| Password reset flow | ❌ Complex (email verification, tokens, expiry) | ✅ Not needed |
| Maintenance | ❌ Must build/maintain password reset, lockout, 2FA | ✅ GitHub handles MFA |

**The Hackathon Context**: HackIn's audience is **developers**. Every developer is on GitHub. OAuth login also validates that the user is a real developer with an actual GitHub account, providing implicit credibility verification.

---

## 👥 Team Features — Deep Dive

### Creating a Team
- `POST /api/v1/team/create-team`
- Team leader creates team with: name, description, teamSize, hackathonName, skills, lookingFor roles
- A **unique 6-character team code** is auto-generated using `crypto.randomBytes(3).toString('hex').toUpperCase()`
- `isLive: true` marks the team as open for recruitment

### Finding a Team (HackMates Page)
- `GET /api/v1/team/get-all` — Lists all active teams
- Users can browse teams by: skills, hackathon name, available roles
- HackMates page displays team cards with "Join Request" button

### Joining a Team
```
User sends join request
      │
      ▼
POST /api/v1/team/join-team
{ teamId, userId, message }
      │
      ▼
Team.joinRequests.push({ userId, message })
User.myRequests.push({ teamId, status: "Pending" })
      │
      ▼
Email sent to team leader (Nodemailer)
"[UserName] wants to join your team [TeamName]"
```

### Accepting a Join Request
```
Team Leader clicks "Accept"
      │
      ▼
POST /api/v1/team/accept-request
{ teamId, userId }
      │
      ▼
Team.teamMembers.push(userId)
Team.joinRequests = filter out the request
User.myRequests → status: "Accepted"
      │
      ▼
Pre-save hook fires:
  isTeamFull = teamMembers.length >= teamSize
  teamScore = sum of all member contributionScores
      │
      ▼
Email sent to applicant:
"Your request to join [TeamName] has been accepted!"
```

### Rejecting a Join Request
```
POST /api/v1/team/reject-request
→ Removes from joinRequests
→ Updates myRequests status to "Rejected"
→ Email sent to applicant
```

### Team Score (Auto-Calculated)
The `Team.pre('save')` middleware automatically:
1. Populates all `teamMembers` documents
2. Sums their `contributionScore` values
3. Sets `teamScore` — used in the Leaderboard

---

## 📧 Email Notifications — How They Work

HackIn uses **Nodemailer** with Gmail SMTP to send transactional emails.

### Configuration
```js
// backend/src/utils/sendmail.js
const transporter = nodemailer.createTransport({
  host: 'smtp.gmail.com',
  port: 587,          // TLS port (not SSL 465)
  secure: false,      // STARTTLS
  auth: {
    user: process.env.EMAIL,
    pass: process.env.PASSWORD  // Gmail App Password (not real password)
  }
});
```

> ⚠️ A **Gmail App Password** must be used (Google account → Security → 2-Step Verification → App Passwords). The actual Gmail password won't work with SMTP.

### Three Email Events

| Event | Trigger | Recipient | Function |
|-------|---------|-----------|----------|
| Join Request Received | User sends request to team | Team Leader | `sendJoinRequest(leaderEmail, leaderName, userName, teamName)` |
| Request Accepted | Leader accepts applicant | Applicant | `sendAcceptMessage(email, userName, teamName)` |
| Request Rejected | Leader rejects applicant | Applicant | `sendRejectMessage(email, userName, teamName)` |

### Email Templates
Currently HTML string templates:
```html
<!-- Accept -->
Hello [UserName],<br>
Your request to join [TeamName] has been accepted.

<!-- Reject -->
Hello [UserName],<br>
Your request to join [TeamName] has been rejected.

<!-- Join Request to Leader -->
Hello [LeaderName],<br>
[UserName] has requested to join your team [TeamName]
```

### Why Nodemailer + Gmail SMTP?
- Zero cost for small-to-medium volume
- Simple setup with `nodemailer.createTransport()`
- For production scale: upgrade to **SendGrid** or **AWS SES** for rate limits and deliverability

---

## 💬 Real-Time Chat — Socket.io

HackIn implements **team-scoped real-time chat** using Socket.io.

### Architecture
```
HTTP Server (Node.js)
      │
      └── Socket.io Server (attached to same HTTP server)
                │
                ├── Event: "join_team" → socket.join(teamId)
                ├── Event: "send_message" → save to DB + io.to(teamId).emit("new_message")
                └── Event: "disconnect" → cleanup
```

### Message Flow
```
Team member opens chat
      │
      ▼
Frontend: socket.emit("join_team", teamId)
      │
      ▼
Server: socket.join(teamId)  // joins the room
      │
      ▼
User types and sends message
      │
      ▼
Frontend: socket.emit("send_message", { teamId, senderId, senderName, content })
      │
      ▼
Server: saves to Message collection (MongoDB persistence)
      │
      ▼
Server: io.to(teamId).emit("new_message", savedMessage)
      │
      ▼
All team members in the room receive the message in real-time
```

