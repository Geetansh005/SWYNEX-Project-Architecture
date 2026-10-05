# FeaturePulse — Full-Stack Application Architecture

<div align="center">

  [![SWYNEX Internship](https://img.shields.io/badge/SWYNEX%20Technologies-Task%201%3A%20Project%20Architecture-0A66C2?style=for-the-badge&logo=codeforces&logoColor=white)](https://github.com/Geetansh005/SWYNEX-Project-Architecture)
  [![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)](https://github.com/Geetansh005/SWYNEX-Project-Architecture)
  [![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)](LICENSE)

  <br />

  [![Next.js](https://img.shields.io/badge/Next.js%2015-000000?style=flat-square&logo=nextdotjs&logoColor=white)](https://nextjs.org/)
  [![React](https://img.shields.io/badge/React%2019-20232A?style=flat-square&logo=react&logoColor=61DAFB)](https://react.dev/)
  [![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
  [![Prisma](https://img.shields.io/badge/Prisma%20ORM-2D3748?style=flat-square&logo=prisma&logoColor=white)](https://www.prisma.io/)
  [![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
  [![Tailwind CSS](https://img.shields.io/badge/Tailwind%20CSS-38B2AC?style=flat-square&logo=tailwind-css&logoColor=white)](https://tailwindcss.com/)

  <p align="center">
    <strong>A modern customer feedback and public roadmap platform designed for agile teams, indie hackers, and SaaS products.</strong>
  </p>

</div>

---

## 📌 Submission Overview

| Attribute | Details |
|---|---|
| **Organization** | **SWYNEX Technologies** |
| **Track** | Full-Stack Web Development Internship |
| **Task** | **Task 1: Project Architecture** |
| **Author** | **Geetansh** ([@Geetansh005](https://github.com/Geetansh005)) |
| **Repository URL** | [https://github.com/Geetansh005/SWYNEX-Project-Architecture](https://github.com/Geetansh005/SWYNEX-Project-Architecture) |

---

## 📖 Table of Contents
1. [Product Overview & Value Proposition](#1-product-overview--value-proposition)
2. [System Architecture](#2-system-architecture)
3. [Data Models & Database Schema](#3-data-models--database-schema)
4. [API Routes & Endpoint Specifications](#4-api-routes--endpoint-specifications)
5. [Frontend Screens & User Flow](#5-frontend-screens--user-flow)
6. [Security, Concurrency & Performance](#6-security-concurrency--performance)
7. [Repository Structure](#7-repository-structure)
8. [Getting Started & Local Setup](#8-getting-started--local-setup)

---

## 1. Product Overview & Value Proposition

### 1.1 Product Idea: FeaturePulse
Software companies frequently struggle with scattered customer feedback coming in through emails, Discord messages, support tickets, and social media.

**FeaturePulse** is an interactive, full-stack feedback and roadmap management web application. It enables companies to host a public feedback board where users can submit feature requests, upvote existing proposals, participate in discussions, and view the real-time development roadmap (*Planned*, *In Progress*, *Completed*).

### 1.2 Target User Personas
* **Product Managers / Founders:** Need a single source of truth to triage incoming feature requests, identify the most requested items by vote volume, and communicate roadmap progress transparently.
* **End Users & Customers:** Want a simple, friction-free portal to submit ideas, upvote features they care about, and receive updates when an idea is moved into active development.

### 1.3 Feature Matrix (MVP vs. Post-MVP)

| Feature | Description | MVP | Post-MVP |
|---|---|:---:|:---:|
| **Public Feedback Wall** | Filterable & sortable list of user feedback items (Trending, Top, New) | ✅ | — |
| **Atomic One-Click Upvoting** | Upvote toggle with client-side optimistic UI and double-vote protection | ✅ | — |
| **Interactive Roadmap** | 3-column Kanban view (*Planned*, *In Progress*, *Completed*) | ✅ | — |
| **Post Submission & Search** | Submission modal with debounced duplicate suggestion detection | ✅ | — |
| **Threaded Comments** | Markdown-enabled discussion on individual feedback items | ✅ | — |
| **Admin Triage Console** | Role-based status updates, pinning, category tagging, and moderation | ✅ | — |
| **Email Status Notifications** | Automated alerts when an upvoted post changes state | ❌ | 🚀 Phase 2 |
| **Custom Domains & Theming** | Subdomain routing (`feedback.customer.com`) with custom branding | ❌ | 🚀 Phase 2 |
| **Linear / Jira Two-Way Sync** | Automatic ticket creation when a post enters *Planned* status | ❌ | 🚀 Phase 2 |

---

## 2. System Architecture

```mermaid
graph TD
    subgraph Client ["Client Layer (Browser)"]
        UI["Next.js App Router (React 19, TypeScript)"]
        Tailwind["Tailwind CSS + Radix UI Primitives"]
        QueryClient["TanStack Query (Optimistic UI State)"]
    end

    subgraph Server ["Application Layer (Node.js / Edge Runtime)"]
        NextServer["Next.js Server Actions & API Route Handlers"]
        AuthMiddleware["NextAuth.js (JWT Sessions & Role Verification)"]
        Validator["Zod Schema Validation Engine"]
    end

    subgraph Data ["Data & Storage Layer"]
        Prisma["Prisma ORM Client"]
        Postgres[("PostgreSQL Database (Neon / Supabase)")]
        Redis[("Upstash Redis (Vote Rate-Limiting & Caching)")]
    end

    UI -->|HTTP / Fetch Requests| NextServer
    NextServer --> AuthMiddleware
    NextServer --> Validator
    NextServer --> Prisma
    Prisma --> Postgres
    NextServer --> Redis
```

### Architectural Decisions
* **Next.js 15 (App Router):** Combines Server-Side Rendering (SSR) for high-performance public board pages and SEO with Server Actions for simplified CRUD mutations.
* **PostgreSQL + Prisma ORM:** Enforces strict relational integrity between users, boards, posts, votes, and comments, with type safety generated directly into TypeScript.
* **Upstash Redis:** Provides sliding-window rate limiting on public voting and submission endpoints to prevent bot abuse.

---

## 3. Data Models & Database Schema

### 3.1 Entity Relationship Diagram (ERD)

```mermaid
erDiagram
    USER ||--o{ BOARD : "owns"
    USER ||--o{ POST : "authors"
    USER ||--o{ VOTE : "casts"
    USER ||--o{ COMMENT : "writes"

    BOARD ||--o{ POST : "contains"
    BOARD ||--o{ CATEGORY : "defines"

    POST ||--o{ VOTE : "receives"
    POST ||--o{ COMMENT : "has"
    POST }o--|| CATEGORY : "categorized_by"

    USER {
        string id PK
        string email UK
        string name
        string avatarUrl
        enum role "USER | ADMIN"
        datetime createdAt
    }

    BOARD {
        string id PK
        string name
        string slug UK
        string description
        string ownerId FK
        boolean isPublic
        datetime createdAt
    }

    CATEGORY {
        string id PK
        string boardId FK
        string name
        string colorHex
    }

    POST {
        string id PK
        string boardId FK
        string authorId FK
        string categoryId FK
        string title
        string description
        enum status "OPEN | UNDER_REVIEW | PLANNED | IN_PROGRESS | COMPLETED | CLOSED"
        int voteCount
        int commentCount
        boolean isPinned
        datetime createdAt
        datetime updatedAt
    }

    VOTE {
        string id PK
        string postId FK
        string userId FK
        datetime createdAt
    }

    COMMENT {
        string id PK
        string postId FK
        string authorId FK
        string content
        boolean isInternal
        datetime createdAt
    }
```

### 3.2 Prisma Schema Specification (`prisma/schema.prisma`)

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

enum Role {
  USER
  ADMIN
}

enum PostStatus {
  OPEN
  UNDER_REVIEW
  PLANNED
  IN_PROGRESS
  COMPLETED
  CLOSED
}

model User {
  id        String    @id @default(cuid())
  email     String    @unique
  name      String?
  avatarUrl String?
  role      Role      @default(USER)
  boards    Board[]   @relation("BoardOwner")
  posts     Post[]
  votes     Vote[]
  comments  Comment[]
  createdAt DateTime  @default(now())
}

model Board {
  id          String     @id @default(cuid())
  name        String
  slug        String     @unique
  description String?
  isPublic    Boolean    @default(true)
  ownerId     String
  owner       User       @relation("BoardOwner", fields: [ownerId], references: [id], onDelete: Cascade)
  categories  Category[]
  posts       Post[]
  createdAt   DateTime   @default(now())

  @@index([slug])
}

model Category {
  id       String @id @default(cuid())
  boardId  String
  board    Board  @relation(fields: [boardId], references: [id], onDelete: Cascade)
  name     String
  colorHex String @default("#6366F1")
  posts    Post[]

  @@unique([boardId, name])
}

model Post {
  id           String     @id @default(cuid())
  boardId      String
  board        Board      @relation(fields: [boardId], references: [id], onDelete: Cascade)
  authorId     String
  author       User       @relation(fields: [authorId], references: [id], onDelete: Cascade)
  categoryId   String?
  category     Category?  @relation(fields: [categoryId], references: [id], onDelete: SetNull)
  title        String
  description  String
  status       PostStatus @default(OPEN)
  voteCount    Int        @default(0)
  commentCount Int        @default(0)
  isPinned     Boolean    @default(false)
  votes        Vote[]
  comments     Comment[]
  createdAt    DateTime   @default(now())
  updatedAt    DateTime   @updatedAt

  @@index([boardId, status])
  @@index([boardId, voteCount(sort: Desc)])
}

model Vote {
  id        String   @id @default(cuid())
  postId    String
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())

  @@unique([postId, userId]) // Guarantees 1 vote per user per post
  @@index([postId])
}

model Comment {
  id         String   @id @default(cuid())
  postId     String
  post       Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  authorId   String
  author     User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  content    String
  isInternal Boolean  @default(false) // Private admin notes
  createdAt  DateTime @default(now())

  @@index([postId, createdAt])
}
```

---

## 4. API Routes & Endpoint Specifications

All endpoints return JSON responses. Standard REST HTTP status codes (`200`, `201`, `400`, `401`, `403`, `404`, `500`) are used.

### 4.1 Route Catalog

| Method | Path | Auth Level | Description |
|---|---|:---:|---|
| `GET` | `/api/boards/:slug` | Public | Get board metadata, settings, and category taxonomy |
| `GET` | `/api/boards/:slug/posts` | Public | Paginated post list (filterable by status, category, sorting) |
| `POST` | `/api/boards/:slug/posts` | User | Submit a new feature request or bug report |
| `GET` | `/api/posts/:id` | Public | Get full post details, vote status, and comments |
| `POST` | `/api/posts/:id/vote` | User | Atomic toggle for upvote / un-vote |
| `POST` | `/api/posts/:id/comments` | User | Post a comment on a feedback thread |
| `PATCH`| `/api/posts/:id/status` | Admin | Update status (*Planned*, *In Progress*, *Completed*) |
| `GET` | `/api/boards/:slug/roadmap`| Public | Fetch grouped posts for the 3 Kanban roadmap columns |

---

### 4.2 Endpoint Deep Dives

#### 1. Retrieve Posts (`GET /api/boards/:slug/posts`)
* **Query Parameters:**
  * `status`: `OPEN` | `UNDER_REVIEW` | `PLANNED` | `IN_PROGRESS` | `COMPLETED` | `ALL`
  * `sort`: `trending` | `top` | `newest`
  * `category`: `string` (category ID)
  * `page`: `number` (default: `1`), `limit`: `number` (default: `20`)

**Response (`200 OK`):**
```json
{
  "data": [
    {
      "id": "cly109a1z0001",
      "title": "Dark Mode Support",
      "description": "Please provide a native dark theme toggle across all dashboard views.",
      "status": "IN_PROGRESS",
      "voteCount": 142,
      "commentCount": 18,
      "hasVoted": true,
      "category": { "id": "cat_ui", "name": "UI/UX", "colorHex": "#8B5CF6" },
      "author": { "name": "Sarah Connor", "avatarUrl": "https://avatar.vercel.sh/sarah" },
      "createdAt": "2026-09-12T10:15:30.000Z"
    }
  ],
  "meta": { "page": 1, "limit": 20, "totalCount": 84, "totalPages": 5 }
}
```

#### 2. Atomic Toggle Vote (`POST /api/posts/:id/vote`)
* **Headers:** `Authorization: Bearer <token>`
* **Execution:** Handled inside an atomic database transaction. If the user has already voted, the vote is deleted and `voteCount` is decremented; otherwise, the vote is created and `voteCount` is incremented.

**Response (`200 OK`):**
```json
{
  "postId": "cly109a1z0001",
  "hasVoted": true,
  "voteCount": 143
}
```

#### 3. Update Status (`PATCH /api/posts/:id/status`)
* **Headers:** `Authorization: Bearer <admin_token>`
* **Request Body:**
```json
{
  "status": "COMPLETED",
  "internalNote": "Released in v2.4.0 update."
}
```
**Response (`200 OK`):**
```json
{
  "id": "cly109a1z0001",
  "status": "COMPLETED",
  "updatedAt": "2026-10-05T12:00:00.000Z"
}
```

---

## 5. Frontend Screens & User Flow

```mermaid
flowchart LR
    A[Public Feedback Board] -->|Click Card| B[Post Detail & Comments Modal]
    A -->|Click '+ Give Feedback'| C[Post Submission Modal]
    A -->|Switch Tab| D[Interactive Roadmap View]
    D -->|Click Card| B
    A -->|Admin Login| E[Admin Triage Dashboard]
    E -->|Drag & Drop| D
```

### Screen 1: Public Feedback Wall (`/b/:slug`)
* **Header:** Board title, search bar, active tab switchers (*Feedback Board*, *Roadmap*), and authentication status.
* **Control Toolbar:** Status filter pills (*All*, *Open*, *Planned*, *In Progress*, *Completed*), category selectors, sort dropdown (*Trending*, *Top Voted*, *Newest*), and primary `+ Give Feedback` button.
* **Post Card Layout:**
  * **Left:** Large prominent upvote pill showing arrow icon and current count. Clicking triggers instant optimistic UI update.
  * **Middle:** Post title, category badge, 2-line truncated description, author avatar, and relative timestamp.
  * **Right:** Speech bubble icon indicating comment count.

### Screen 2: Post Detail & Discussion Modal (`/b/:slug/posts/:id`)
* Modal with backdrop blur supporting deep-linkable URLs.
* Full markdown description display.
* Real-time status pill (e.g. *In Progress* highlighted in amber).
* Threaded comment timeline displaying user replies with official "Team / Admin" badges for company staff.

### Screen 3: Interactive Roadmap View (`/b/:slug/roadmap`)
* 3-column Kanban layout:
  1. **Planned:** Validated ideas prioritized for development.
  2. **In Progress:** Features currently under active implementation.
  3. **Completed:** Deployed features with changelog notes.
* Admin accounts can drag cards between columns to change statuses directly.

### Screen 4: Feedback Submission Modal (`/b/:slug/new`)
* Live duplicate suggestion: Typing a title performs a debounced search on existing posts and displays: *"Are you asking for this? Upvote instead to consolidate votes!"*
* Fields: Title (single line), Category (dropdown), Details (rich markdown textarea).

### Screen 5: Admin Triage & Moderation Console (`/admin/:slug`)
* Bulk status change controls.
* Private internal notes field visible only to admins.
* Spam moderation tools and CSV export for product analytics.

---

## 6. Security, Concurrency & Performance

* **Vote Deduplication & Concurrency:** A database-level composite unique constraint `@@unique([postId, userId])` guarantees that race conditions cannot lead to duplicate votes per user.
* **Rate Limiting:** Sliding-window rate limiting via Redis prevents rapid vote manipulation or spam submissions (max 30 requests/minute per IP).
* **XSS Sanitization:** All user-submitted markdown content is sanitized on the server before rendering using DOMPurify.
* **Caching (ISR):** Public boards and roadmap columns utilize Next.js Incremental Static Regeneration (revalidated every 60 seconds) for near-instant page loads.

---

## 7. Repository Structure

```
SWYNEX-Project-Architecture/
├── prisma/
│   └── schema.prisma         # Complete PostgreSQL schema & relations
├── src/
│   ├── app/
│   │   ├── api/              # Route handlers for boards, posts, votes, comments
│   │   ├── b/[slug]/         # Public feedback wall & post detail routes
│   │   └── admin/            # Admin moderation & triage dashboard
│   ├── components/
│   │   ├── feedback/         # PostCard, UpvoteButton, PostModal
│   │   ├── roadmap/          # RoadmapBoard, KanbanColumn
│   │   └── ui/               # Radix UI primitives & Tailwind components
│   └── lib/
│       ├── prisma.ts         # Prisma client singleton
│       └── rate-limit.ts     # Upstash Redis rate limiter
├── package.json              # Project dependencies & scripts
├── tailwind.config.ts        # Design tokens & color system
├── .gitignore                # Standard repository ignore list
└── README.md                 # Complete Architecture & Specification Document
```

---

## 8. Getting Started & Local Setup

### Prerequisites
* Node.js (v18.17+ or v20+)
* PostgreSQL database instance (local or hosted via Supabase/Neon)

### Installation Steps

```bash
# 1. Clone the repository
git clone https://github.com/Geetansh005/SWYNEX-Project-Architecture.git
cd SWYNEX-Project-Architecture

# 2. Install dependencies
npm install

# 3. Configure environment variables in a .env file
# DATABASE_URL="postgresql://username:password@localhost:5432/featurepulse"
# NEXTAUTH_SECRET="your-super-secret-key"

# 4. Generate Prisma client & sync database
npx prisma generate
npx prisma db push

# 5. Start development server
npm run dev
```

Visit `http://localhost:3000` to view the application.

---

<div align="center">
  <sub>Submitted as part of the <strong>SWYNEX Technologies Internship</strong> | Designed & Developed by <strong>Geetansh</strong></sub>
</div>
