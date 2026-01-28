# Port Moresby National High School Web Application

## Overview

This is a full-stack web application for Port Moresby National High School, serving as both a public-facing school website and an internal academic management system. The application supports three user types: general public (unauthenticated visitors), students, and teachers/administrators.

**Core Features:**
- Public pages: Home, About, Academics, Staff, News, Contact
- Role-based dashboard system (Student vs Teacher views)
- Academic tracking: GPA, grades, attendance records
- File management system
- Real-time messaging between users
- Community feed with posts and events
- Peer/colleague connections

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend Architecture
- **Framework:** React 18 with TypeScript
- **Routing:** Wouter (lightweight React router)
- **State Management:** TanStack React Query for server state caching and synchronization
- **Styling:** Tailwind CSS with shadcn/ui component library (New York style variant)
- **Build Tool:** Vite with path aliases (@/, @shared/, @assets/)

**Key Design Decisions:**
- Component organization follows feature-based structure (components/modules/, components/dashboard/, components/public/)
- Protected routes redirect unauthenticated users to Replit Auth login
- Real-time updates via WebSocket for teachers/admins only

### Backend Architecture
- **Runtime:** Node.js with Express.js
- **Language:** TypeScript (ESM modules)
- **API Pattern:** RESTful endpoints defined in shared/routes.ts with Zod validation schemas
- **Authentication:** Replit Auth (OpenID Connect) with session storage in PostgreSQL

**Key Design Decisions:**
- Shared route definitions between client and server for type safety
- Session management via connect-pg-simple storing sessions in database
- WebSocket server for real-time data broadcasting to privileged users

### Data Storage
- **Database:** PostgreSQL via Drizzle ORM
- **Schema Location:** shared/schema.ts (shared between frontend and backend)
- **Migrations:** Generated via `drizzle-kit push` command

**Database Tables:**
- `users` - User accounts (managed by Replit Auth)
- `sessions` - Session storage (required for Replit Auth)
- `user_roles` - Role assignments (student/teacher/admin)
- `academic_records` - GPA, attendance, class rank
- `subjects` - Course/subject definitions
- `grades` - Student grades per subject
- `files` - File metadata storage
- `messages` - User-to-user messaging
- `community_posts` - Community feed posts
- `connections` - Peer/colleague relationships

### Build & Development
- **Dev Server:** tsx for TypeScript execution, Vite for frontend HMR
- **Production Build:** Custom build script using esbuild (server) + Vite (client)
- **Output:** dist/index.cjs (server) and dist/public/ (static assets)

## External Dependencies

### Authentication
- **Replit Auth:** OpenID Connect integration via `openid-client` library
- Requires `ISSUER_URL`, `REPL_ID`, and `SESSION_SECRET` environment variables

### Database
- **PostgreSQL:** Required, connection via `DATABASE_URL` environment variable
- **Drizzle ORM:** Schema-first approach with PostgreSQL dialect

### UI Component Library
- **shadcn/ui:** Pre-built accessible components using Radix UI primitives
- Configuration in components.json with path aliases

### Key Runtime Dependencies
- `@tanstack/react-query` - Data fetching and caching
- `drizzle-orm` / `drizzle-zod` - Database ORM and validation
- `express-session` / `connect-pg-simple` - Session management
- `ws` - WebSocket server for real-time features
- `date-fns` - Date formatting utilities
- `lucide-react` - Icon library

## Real-Time Updates (Teacher/Admin Only)

The application includes WebSocket-based real-time updates for teacher and admin users. When any data is inserted, updated, or deleted through the API, connected teacher/admin clients receive automatic notifications and refresh their data.

**Implementation:**
- `server/websocket.ts` - WebSocket server that manages connections and broadcasts data changes
- `client/src/hooks/use-realtime.ts` - React hook that connects to WebSocket and invalidates React Query cache on updates
- Routes in `server/routes.ts` call `broadcastDataChange()` after mutations

**Supported Resources:**
- Files (create, delete)
- Messages (create)
- Community Posts (create)

**How it works:**
1. Teacher/Admin logs in and loads the Dashboard
2. The `useRealtime` hook connects to `/ws` WebSocket endpoint
3. When any user makes changes (create/delete files, send messages, create posts), the server broadcasts the change
4. Connected teacher/admin clients automatically refresh the affected data