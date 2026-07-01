# Complete Setup & Installation Guide

This guide walks you through setting up the Anime Generator app from scratch.

## 📋 Prerequisites

Before starting, make sure you have:

- **Git** installed - [Download](https://git-scm.com)
- **Node.js 18+** - [Download](https://nodejs.org)
- **Docker Desktop** - [Download](https://www.docker.com/products/docker-desktop)
- **A GitHub account** - [Sign up](https://github.com)
- **Replicate account** - [Sign up free](https://replicate.com)

Verify installations:
```bash
git --version
node --version
npm --version
docker --version
```

---

## Step 1: Clone the Repository

Open your terminal and run:

```bash
git clone https://github.com/jsagar1997-sys/anime-generator-video-and-photos.git
cd anime-generator-video-and-photos
```

Verify you're on the `initial-setup` branch:
```bash
git branch
```

You should see:
```
* initial-setup
  master
```

---

## Step 2: Setup Environment Variables

### 2.1: Create `.env` file

Copy the example file:
```bash
cp .env.example .env
```

### 2.2: Get Your Replicate API Token

1. Go to [https://replicate.com](https://replicate.com)
2. Sign up or log in
3. Click your profile → **Account** (or go to https://replicate.com/account)
4. Under "API tokens", copy your token (looks like: `r8_xxxxxxxxxxxxxxxxxxxxxxxxxxxx`)

### 2.3: Update `.env` file

Open `.env` in your editor and add your Replicate token:

```bash
# Backend
PORT=3001
DATABASE_URL=postgresql://anime_user:anime_password@localhost:5432/anime_generator
REDIS_URL=redis://localhost:6379
REPLICATE_API_TOKEN=r8_your_actual_token_here  # ← Paste your token here
JWT_SECRET=your_super_secret_jwt_key_change_this
NODE_ENV=development

# Frontend
VITE_API_URL=http://localhost:3001

# AI Services
STABLE_DIFFUSION_API=https://api.replicate.com
```

**⚠️ Important**: Never commit `.env` to Git. It's already in `.gitignore`.

---

## Step 3: Start Docker Services

Docker containers run your database and cache.

### 3.1: Start Docker Desktop

Open Docker Desktop application (it runs in background).

### 3.2: Start Services

In your terminal, run:
```bash
npm run docker:up
```

You'll see:
```
Creating anime-db ... done
Creating anime-cache ... done
```

### 3.3: Verify Services are Running

Check Docker:
```bash
docker ps
```

You should see two containers:
- `anime-db` (PostgreSQL)
- `anime-cache` (Redis)

Test connection:
```bash
# Test PostgreSQL
psql -h localhost -U anime_user -d anime_generator -c "SELECT version();"

# Test Redis (if redis-cli installed)
redis-cli ping  # Should return: PONG
```

---

## Step 4: Install Dependencies

Install npm packages for all packages:

```bash
npm install
```

This installs dependencies for:
- Backend
- Web app
- Desktop app
- Mobile app
- Shared types

**Time**: This may take 3-5 minutes. ☕

---

## Step 5: Start Development Servers

You'll need to open **4 separate terminals** (or use terminal tabs).

### Terminal 1: Backend API Server

```bash
cd packages/backend
npm run dev
```

Expected output:
```
🚀 Server running on http://localhost:3001
📊 API available at http://localhost:3001/api/v1
```

### Terminal 2: Web Frontend

```bash
cd packages/web
npm run dev
```

Expected output:
```
VITE v4.3.8  ready in 123 ms

➜  Local:   http://localhost:3000/
➜  press h to show help
```

### Terminal 3: Desktop App (Optional)

```bash
cd packages/desktop
npm run dev
```

### Terminal 4: Mobile App (Optional)

```bash
cd packages/mobile
npm run dev
```

---

## Step 6: Access Your Applications

Open your browser:

### 🌐 Web App
```
http://localhost:3000
```

You should see the Anime Generator homepage with:
- "Start Generating" button
- "View Gallery" button
- Feature cards

### 📡 API Health Check
```
http://localhost:3001/health
```

Response:
```json
{
  "status": "ok",
  "timestamp": "2024-07-01T12:00:00.000Z"
}
```

### 📊 API Status
```
http://localhost:3001/api/v1/status
```

Response:
```json
{
  "service": "anime-generator-api",
  "version": "1.0.0",
  "timestamp": "2024-07-01T12:00:00.000Z"
}
```

---

## Step 7: Test Image Generation

### 7.1: Via Web Interface

1. Go to http://localhost:3000
2. Click "Start Generating"
3. Select "Image" type
4. Enter prompt: `"anime girl with blue hair, smiling, kawaii style"`
5. Click "Generate"

You should see:
```
Status: queued
Job ID: job_123
Progress: 0%
```

Click "Check Status" to update progress.

### 7.2: Via API (cURL)

```bash
curl -X POST http://localhost:3001/api/v1/generate/image \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "anime girl with blue hair",
    "userId": "test-user",
    "style": "anime"
  }'
```

Response:
```json
{
  "jobId": "1",
  "status": "queued",
  "message": "Image generation queued successfully"
}
```

### 7.3: Check Job Status

```bash
curl http://localhost:3001/api/v1/job/1
```

Response:
```json
{
  "jobId": "1",
  "state": "pending",
  "progress": 0,
  "data": {
    "prompt": "anime girl with blue hair",
    "userId": "test-user",
    "style": "anime"
  },
  "result": null
}
```

---

## Step 8: Troubleshooting

### ❌ PostgreSQL Connection Error

**Error**: `ECONNREFUSED 127.0.0.1:5432`

**Solution**:
```bash
# Check if Docker is running
docker ps

# Restart Docker services
npm run docker:down
npm run docker:up

# Check logs
docker logs anime-db
```

### ❌ Redis Connection Error

**Error**: `Error: connect ECONNREFUSED 127.0.0.1:6379`

**Solution**:
```bash
# Verify Redis is running
docker ps | grep anime-cache

# Restart
docker restart anime-cache

# Test connection
redis-cli -h localhost ping
```

### ❌ Port Already in Use

**Error**: `Port 3000 is already in use` or `Port 3001 is already in use`

**Solution**:
```bash
# Find process on port
lsof -i :3000  # for web
lsof -i :3001  # for backend

# Kill process (replace PID)
kill -9 <PID>

# Or use different ports
PORT=3002 npm run dev  # for backend
```

### ❌ npm install fails

**Error**: `npm ERR! code ERESOLVE`

**Solution**:
```bash
# Clear npm cache
npm cache clean --force

# Install with legacy peer deps
npm install --legacy-peer-deps

# Or use npm 7+
npm install
```

### ❌ Module not found errors

**Error**: `Cannot find module 'express'`

**Solution**:
```bash
# Make sure you're in correct directory
cd packages/backend
npm install

# Then try again
npm run dev
```

---

## Step 9: Understanding the Architecture

```
┌─────────────────────────────────────────┐
│         Your Browser/Desktop            │
│  • Web: localhost:3000 (React)          │
│  • Mobile: Expo app                     │
│  • Desktop: Electron app                │
└──────────────────┬──────────────────────┘
                   │ HTTP/WebSocket
                   ↓
┌─────────────────────────────────────────┐
│    Backend API (Node.js/Express)        │
│    localhost:3001                       │
│  • /api/v1/generate/image               │
│  • /api/v1/generate/video               │
│  • /api/v1/job/:jobId                   │
└──────────┬───────────────────────┬──────┘
           │                       │
      ┌────↓────┐         ┌───────↓──────┐
      │          │         │              │
      ↓          ↓         ↓              ↓
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│PostgreSQL│ │  Redis   │ │ Bull Job │ │Replicate │
│Database  │ │  Cache   │ │  Queue   │ │   API    │
└──────────┘ └──────────┘ └──────────┘ └──────────┘
```

---

## Step 10: Next Steps

### ✅ You've Successfully Setup:
- [x] Backend API server
- [x] Web frontend
- [x] Database (PostgreSQL)
- [x] Cache (Redis)
- [x] Job queue (Bull)

### 🔄 What to Do Next:

1. **Add Authentication** (Login/Signup)
   - See `packages/backend/src/auth.ts` (to be created)

2. **Setup Replicate Integration**
   - Configure image/video generation workers

3. **Create Pull Request**
   - Merge `initial-setup` branch to `main`

4. **Deploy to Production**
   - Deploy backend to Railway/Render
   - Deploy frontend to Vercel/Netlify
   - Setup managed database

5. **Add More Features**
   - User profiles
   - Gallery/history
   - Favorites
   - Sharing

---

## Step 11: Useful Commands

### Development
```bash
# Start all dev servers (from root)
npm run dev

# Start specific service
cd packages/backend && npm run dev
cd packages/web && npm run dev

# Build for production
npm run build

# Run tests
npm test
```

### Docker
```bash
# Start services
npm run docker:up

# Stop services
npm run docker:down

# View logs
docker logs anime-db
docker logs anime-cache

# Clean everything
docker system prune
```

### Database
```bash
# Connect to PostgreSQL
psql -h localhost -U anime_user -d anime_generator

# List tables
\dt

# Exit
\q
```

---

## Step 12: File Structure Explained

```
packages/backend/
├── src/
│   └── index.ts           # Main server file
├── package.json           # Dependencies
└── README.md              # Backend docs

packages/web/
├── src/
│   ├── App.tsx            # Main React component
│   ├── pages/
│   │   ├── Home.tsx       # Homepage
│   │   ├── Generator.tsx  # Generation page
│   │   └── Gallery.tsx    # Gallery page
│   ├── components/
│   │   └── Header.tsx     # Navigation
│   ├── main.tsx           # Entry point
│   ├── index.css          # Styles
├── index.html             # HTML template
├── vite.config.ts         # Vite config
└── package.json           # Dependencies

packages/desktop/
├── src/
│   └── main.ts            # Electron main process

packages/mobile/
├── app.json               # Expo config

packages/shared/
├── src/
│   └── types.ts           # Shared TypeScript types
```

---

## Congratulations! 🎉

You now have a fully functional anime generation application with:
- ✅ Backend API running
- ✅ Web frontend ready
- ✅ Database configured
- ✅ Ready to integrate AI models
- ✅ Desktop & mobile ready

**Next**: Follow [REPLICATE_SETUP.md](./REPLICATE_SETUP.md) to integrate image/video generation.
