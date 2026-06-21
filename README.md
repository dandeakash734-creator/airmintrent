# Marginal — Social Media App Prototype

A full-stack social media prototype: feed, posts (text/image), likes, comments,
follow/profile, 24-hour stories, and direct messages.

**Stack**
- Backend: Node.js, Express, PostgreSQL, Prisma, JWT auth
- Frontend: React (Vite), React Router
- Image storage: any S3-compatible bucket (AWS S3, Cloudflare R2, MinIO, etc.)

This was built and syntax-checked in a sandboxed environment without network
access, so it has **not** been run end-to-end yet. Follow the steps below on
your own machine to get it running — they're standard for this stack, but
test carefully and let me know if anything doesn't line up.

## 1. Prerequisites

- Node.js 18+ and npm
- A PostgreSQL database (local install, or a free hosted one like
  [Neon](https://neon.tech) or [Supabase](https://supabase.com))
- An S3-compatible bucket and credentials (AWS S3, Cloudflare R2, or run
  [MinIO](https://min.io) locally for free if you just want to test)

## 2. Backend setup

```bash
cd backend
npm install
cp .env.example .env
```

Open `.env` and fill in:
- `DATABASE_URL` — your PostgreSQL connection string
- `JWT_SECRET` — any long random string
- `S3_*` — your bucket credentials

Then create the database tables and start the server:

```bash
npx prisma migrate dev --name init
npm run dev
```

The API will run at `http://localhost:4000`. Check `http://localhost:4000/health`
in your browser — it should return `{"status":"ok"}`.

## 3. Frontend setup

In a new terminal:

```bash
cd frontend
npm install
cp .env.example .env
npm run dev
```

Open `http://localhost:5173`. Register an account, and you're in.

## 4. Trying it out

- Create a second account in an incognito window to test follow, likes,
  comments, stories, and DMs between two users.
- Stories disappear after 24 hours automatically (filtered by `expiresAt` in
  the API — no cron job needed for a prototype).
- If your feed is empty (no one to follow yet), the app falls back to an
  "explore" view of all posts so it isn't blank.

## Project structure

```
backend/
  prisma/schema.prisma   — database models
  src/routes/            — auth, users, posts, stories, messages
  src/middleware/        — JWT auth, file upload, error handling
  src/server.js          — entry point
frontend/
  src/pages/             — Feed, Profile, Post detail, Messages, Settings, Auth
  src/components/        — NavBar, PostCard, ComposeBox, StoriesRail, etc.
  src/context/           — auth state
  src/api/               — API client functions
  src/styles/            — design system CSS
```

## Notes on what's intentionally left out (it's a prototype)

- No real-time updates — messages and feeds are fetched on load/action, not
  pushed via WebSockets. Worth adding if you take this further.
- No rate limiting, email verification, or password reset flow.
- No pagination UI on the frontend yet, even though the feed/explore API
  supports a `page` query param.
- Story expiry is enforced at query time, not by deleting old rows — fine for
  a prototype, but you'd want a cleanup job in production.

## If something doesn't work

Since I couldn't run this myself before handing it off, the most likely
friction points are:
1. `DATABASE_URL` formatting — double check user/password/host/port/dbname.
2. S3 bucket permissions — the upload code sets `ACL: 'public-read'`; some
   providers (like R2) don't support ACLs the same way and may need a public
   bucket policy instead, or a custom public base URL.
3. CORS — `CLIENT_URL` in the backend `.env` must exactly match the frontend's
   origin (including port).

Paste me any error message and I can help debug it.
