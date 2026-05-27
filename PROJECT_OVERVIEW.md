# Toucher: Project Overview & Specification

This document serves as the primary reference for the Toucher lawn bowls management system.

## 🌟 Vision
Toucher is a multi-tenant platform designed to modernize lawn bowls for clubs of all sizes. It provides a seamless, real-time connection between the physical game on the green and the administrative needs of the clubhouse, while allowing multiple distinct clubs to manage their memberships and tournaments independently within the same ecosystem.

## 🏗️ Technology Stack

Toucher utilizes a unified TypeScript ecosystem with a multi-tenant architecture to maximize code sharing and development velocity.

### 1. Mobile App (The Green)
- **Framework:** React Native + Expo
- **Styling:** **NativeWind** (Tailwind CSS for React Native)
- **Role:** The primary interface for players, markers, and umpires.

### 2. Web Interface (The Clubhouse)
- **Framework:** Next.js (React)
- **Styling:** **Tailwind CSS** + **Shadcn UI** (for accessible, polished components)
- **Role:** The administrative hub and public-facing portal.

### 3. Backend & Data Layer
- **Database:** PostgreSQL (Neon)
- **Primary API:** Next.js API Routes + Prisma
- **Content API:** Strapi (Headless CMS)

### 5. Supplemental Ecosystem (Recommended)
To maintain high performance and type safety across the stack, the following libraries are integrated:
- **Data Fetching:** **TanStack Query (React Query)** — Essential for caching API data and handling real-time synchronization states between the apps and the database.
- **Validation:** **Zod** — For "Schema-first" development. Ensures that data submitted from the mobile app matches the Prisma database schema exactly.
- **Forms:** **React Hook Form** — For handling complex club registrations and tournament setups with high performance.
- **Monorepo Management:** **Turborepo** — To manage the Next.js, Expo, and Strapi projects in a single repository, allowing for shared TypeScript types and logic.
- **Icons:** **Lucide React / Lucide React Native** — A consistent, beautiful icon set for all interfaces.
- **Date Handling:** **date-fns** — For managing tournament schedules, match times, and membership renewals.

### 4. Authentication & Authorization
- **Provider:** Clerk
- **Authentication Methods:**
    - **Passwordless:** Magic Email Links (Primary for frictionless login).
    - **Social (OAuth):** Google and Facebook.
- **Mobile Auth Flow (Expo):**
    - **Session Management:** Uses `@clerk/clerk-expo` to handle secure login, token storage, and session persistence.
    - **Implementation:** Supports Magic Link redirects and native social login flows for Google and Facebook.
    - **Authorization:** For every request to the Next.js API, the Expo app attaches a JWT (Session Token) in the `Authorization: Bearer <token>` header.
- **Web Auth Flow (Next.js):**
    - Uses `@clerk/nextjs` for middleware-based protection of API routes and dashboard pages.
- **Backend Validation & RBAC:**
    - **Token Verification:** The Next.js API validates the JWT using Clerk's backend SDK.
    - **Role-Based Access Control (RBAC):** Next.js checks the user's role (e.g., Player, Marker, Admin) and `clubId` (multi-tenancy) against the database via Prisma before allowing any data modification.
    - **Tenant Isolation:** Ensures a user can only access data belonging to their specific club.

## 📊 Architectural Diagram

```text
            [ Auth: Clerk ]
             (JWT Issuance)
                   │
    ┌──────────────┼──────────────┐
    ▼              ▼              ▼
[ Mobile App ] [ Web Dashboard ] [ Strapi Admin ]
 Expo + JWT      Next.js + JWT     Staff Auth
    │              │              │
    └──────┬───────┴───────┬──────┘
           ▼               ▼
    [ Next.js API ] <──> [ Strapi ]
     (JWT Validated)    (Content API)
           │               │
    └──────┴───────┬───────┴──────┘
                   ▼
         [ Backend & Database ]
          Neon (Postgres) + 
          Prisma ORM
```

## 🔄 Core Workflows

### Tournament Management
1. **Creation:** Admin defines tournament type, dates, and entry requirements via Next.js.
2. **Draw:** The system generates brackets or round-robin schedules.
3. **Check-in:** Players check in via the Mobile App (possibly via QR code).

### Live Scoring
1. **Match Start:** Players/Markers open the match on the Mobile App.
2. **End-by-End Scoring:** Scores are entered after each end.
3. **Instant Sync:** The real-time service broadcasts the update to the Database and the Clubhouse Dashboard.
4. **Conclusion:** Match results are finalized and brackets auto-update.
