# Project Ledger Quick Start Guide

## Development Environment Setup

### Option 1: Using Dev Container (Recommended)

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop)
2. Install [VS Code](https://code.visualstudio.com/)
3. Install the [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack) extension pack in VS Code
4. Open the project in VS Code
5. When prompted "Reopen in Container", click "Reopen in Container". Alternatively, press F1 and select "Dev Containers: Reopen in Container"
6. Wait for the container to build and start (this may take a few minutes the first time)

The dev container includes:
- Node.js 18
- PostgreSQL 14
- All necessary development tools and extensions

### Database Setup

1. Start PostgreSQL container:
```bash
docker-compose up -d
```

2. Wait a few seconds for the database to initialize

Once the database is running, proceed with the migrations:

```bash
cd project-ledger/server
npx prisma migrate dev
```

This will:
1. Create or update the database schema
2. Generate Prisma Client
3. Create and apply migrations
4. Seed the database with initial data

### Running the Application

1. Start the backend server:
```bash
cd project-ledger/server
npm run dev
```

2. In a new terminal, start the frontend:
```bash
cd project-ledger
npm run dev
```

The application will be available at:
- Frontend: http://localhost:3000
- Backend API: http://localhost:3001

## Default Admin Account

After database setup, you can log in with these credentials:
- Email: admin@projectledger.com
- Password: admin123

## Running with Docker Compose

You can also run the entire application stack using Docker Compose:

1. Build and start all services:
```bash
docker-compose up --build
```

This will start:
- Frontend client at http://localhost:3000
- Backend API server at http://localhost:3001
- PostgreSQL database at localhost:5432

2. Initial database setup (first time only):
```bash
# In a new terminal
docker-compose exec server npx prisma migrate dev
```

To stop all services:
```bash
docker-compose down
```

To stop all services and remove volumes:
```bash
docker-compose down -v
```