# Docker Memory Configuration Guide

## Overview
This guide explains how to configure Docker memory settings to prevent out-of-memory issues during development, particularly with the frontend webpack compilation.

## Current Memory Allocation

### Frontend Container
- **Memory Limit**: 6GB (to handle webpack compilation)
- **Memory Reservation**: 2GB (guaranteed allocation)
- **CPU Limit**: 2 cores
- **Node.js Heap**: 4GB (`--max-old-space-size=4096`)

### Backend Container
- **Memory Limit**: 3GB
- **Memory Reservation**: 1GB
- **CPU Limit**: 1.5 cores
- **Node.js Heap**: 2GB (`--max-old-space-size=2048`)

### Database Container
- **Memory Limit**: 2GB
- **Memory Reservation**: 512MB
- **CPU Limit**: 1 core

## Docker Desktop Settings

### Windows/Mac Docker Desktop
1. Open **Docker Desktop**
2. Go to **Settings** → **Resources**
3. Set **Memory** to at least **12GB** (recommended 16GB)
4. Set **CPUs** to at least **4 cores** (recommended 6-8)
5. Click **Apply & Restart**

### Linux Docker Engine
Add to `/etc/docker/daemon.json`:
```json
{
  "default-ulimits": {
    "memlock": {
      "Hard": -1,
      "Name": "memlock",
      "Soft": -1
    }
  },
  "storage-driver": "overlay2"
}
```

## Memory Issues Troubleshooting

### Symptoms
- "JavaScript heap out of memory" errors
- "Process exited [SIGABRT]" messages
- Frontend container crashes during compilation
- 404 errors due to failed webpack builds

### Solutions

#### 1. Increase Docker Desktop Memory
- Minimum: 12GB total
- Recommended: 16GB total
- For heavy development: 20GB+

#### 2. Restart Containers After Memory Changes
```bash
docker compose down
docker compose up -d --build
```

#### 3. Monitor Memory Usage
```bash
# Check container memory usage
docker stats

# Check specific container
docker stats projectledger2-frontend-1
```

#### 4. Clear Docker Cache
```bash
# Clear build cache
docker builder prune

# Clear all unused resources
docker system prune -a
```

## Environment Variables

### Node.js Memory Settings
- `NODE_OPTIONS=--max-old-space-size=4096` (Frontend)
- `NODE_OPTIONS=--max-old-space-size=2048` (Backend)

### Webpack Optimization
- `CHOKIDAR_USEPOLLING=true` (File watching)
- `WATCHPACK_POLLING=true` (Webpack polling)

## Production Considerations

For production deployments, adjust memory based on actual usage:
- Frontend: 1-2GB (built static files)
- Backend: 1-2GB (API server)
- Database: 2-4GB (depends on data size)

## Alternative: Development Without Docker

If memory constraints persist, you can run development locally:

```bash
# Backend
cd apps/backend
npm install
npm run dev

# Frontend (separate terminal)
cd apps/frontend
npm install
npm start
```

Set environment variables in `.env.local` files for local development.