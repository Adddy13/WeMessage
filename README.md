# WeMessage

A real-time, iMessage-styled chat application built with React and Express. Sign in, see who's online, and exchange text, image, and video messages instantly over WebSockets.

## Features

- **Real-time messaging** — powered by Socket.IO; messages arrive instantly for online recipients
- **Online presence** — see which of your contacts are currently online
- **Media messages** — send images and videos alongside text, uploaded via ImageKit
- **Authentication** — handled by Clerk, with a webhook that syncs users into the app's own database
- **Conversation list** — sidebar shows your existing conversations sorted by most recent activity, plus all other users you haven't messaged yet
- **Theming & wallpapers** — light/dark theme presets and a choice of chat wallpapers
- **Keystroke sounds** — optional typing sound effects for that classic messaging feel
- **Dockerized deployment** — a single multi-stage Dockerfile builds the SPA and API into one production image

## Tech Stack

**Frontend**

- React 19 + Vite
- Tailwind CSS + HeroUI / React Aria Components
- Zustand for state management
- React Router
- Socket.IO client
- Clerk for authentication UI

**Backend**

- Node.js + Express 5
- MongoDB + Mongoose
- Socket.IO for real-time events
- Clerk (server SDK) for auth verification + webhooks
- ImageKit for media storage/CDN
- `node-cron` for a periodic keep-alive ping (useful on free-tier hosts that spin down on inactivity)

## Project Structure

```
ChatApp/
├── backend/
│   ├── src/
│   │   ├── controllers/      # Route handlers (auth, messages)
│   │   ├── lib/              # DB connection, socket server, ImageKit, cron job
│   │   ├── middleware/       # Auth guard, file upload handling
│   │   ├── models/           # Mongoose schemas (User, Message)
│   │   ├── routes/           # Express routers
│   │   ├── webhooks/         # Clerk webhook handler
│   │   └── index.js          # App entry point
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/       # UI components (chat, auth, theming)
│   │   ├── context/           # Theme & wallpaper providers
│   │   ├── hooks/
│   │   ├── lib/
│   │   ├── pages/
│   │   └── store/             # Zustand stores
│   └── package.json
├── Dockerfile
└── README.md
```

## Prerequisites

- Node.js 22+
- A MongoDB database (e.g. [MongoDB Atlas](https://www.mongodb.com/atlas))
- A [Clerk](https://clerk.com) application (for authentication)
- An [ImageKit](https://imagekit.io) account (for image/video uploads — optional; the app runs without it, but media uploads will be disabled)

## Getting Started

### 1. Clone and install dependencies

```bash
git clone <your-repo-url>
cd ChatApp

cd backend && npm install
cd ../frontend && npm install
```

### 2. Configure environment variables

**`backend/.env`**

```env
PORT=3000
NODE_ENV=development

MONGO_URI=your_mongodb_connection_string

CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
CLERK_SECRET_KEY=your_clerk_secret_key
CLERK_WEBHOOK_SIGNING_SECRET=your_clerk_webhook_signing_secret

IMAGEKIT_PRIVATE_KEY=your_imagekit_private_key

FRONTEND_URL=http://localhost:5173
```

**`frontend/.env`**

```env
VITE_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
```

> ⚠️ **Never commit real `.env` values to version control.** Rotate any keys that may have been exposed, and add `.env` to `.gitignore` (already included here).

### 3. Set up the Clerk webhook

In your Clerk dashboard, add a webhook pointing to:

```
<your_backend_url>/api/webhooks/clerk
```

subscribed to user creation events, and use the resulting signing secret as `CLERK_WEBHOOK_SIGNING_SECRET`. This keeps your MongoDB `User` collection in sync with Clerk.

### 4. Run in development

```bash
# Terminal 1 — backend (http://localhost:3000)
cd backend
npm run dev

# Terminal 2 — frontend (http://localhost:5173)
cd frontend
npm run dev
```

Open `http://localhost:5173` in your browser.

## API Overview

All routes below are prefixed with `/api` and (aside from the webhook) require a valid Clerk session.

| Method | Endpoint                  | Description                                  |
| ------ | ------------------------- | -------------------------------------------- |
| GET    | `/auth/check`             | Verify the current session                   |
| GET    | `/messages/users`         | List all other users                         |
| GET    | `/messages/conversations` | List conversations, most recent first        |
| GET    | `/messages/:id`           | Get message history with a specific user     |
| POST   | `/messages/send/:id`      | Send a message (text and/or media) to a user |
| POST   | `/webhooks/clerk`         | Clerk webhook receiver (no auth)             |
| GET    | `/health`                 | Health check                                 |

Real-time events are exchanged over Socket.IO: `getOnlineUsers` broadcasts the list of connected user IDs, and `newMessage` is emitted to a recipient the moment a message is sent to them while they're online.

## Building for Production

The included `Dockerfile` builds the whole app (frontend + backend) into a single image: it compiles the Vite SPA, bundles the Express API, and serves the static frontend directly from Express.

```bash
docker build \
  --build-arg VITE_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key \
  -t wemessage .

docker run -p 3001:3001 --env-file backend/.env wemessage
```

The container listens on port `3001` and serves both the API (`/api/*`) and the frontend from the same origin.

## License

No license specified — add one if you intend to share or open-source this project.
