# Exness Trading Platform

A full-stack real-time trading platform built with Next.js, Node.js, WebSockets, and Redis. Live BTC/USDT price data is streamed from Binance and delivered to the browser in real time.

[![Build & Deploy to EC2](https://github.com/jeetupal31/exness/actions/workflows/deploy.yml/badge.svg)](https://github.com/jeetupal31/exness/actions/workflows/deploy.yml)

---

## Live Links

| Service | URL |
|---------|-----|
| **Frontend** | [http://3.110.62.127:3000](http://3.110.62.127:3000) |
| **REST API** | [http://3.110.62.127:4000/api/v1](http://3.110.62.127:4000/api/v1) |
| **WebSocket** | `ws://3.110.62.127:8080` |

---

## Architecture

```
                        Browser
                           |
              ┌────────────┴────────────┐
              │                         │
         HTTP (4000)              WS (8080)
              │                         │
        ┌─────┴──────┐          ┌───────┴──────┐
        │  api-server │          │  ws-server   │
        │  (Express)  │          │  (WebSocket) │
        └─────────────┘          └───────┬──────┘
                                         │
                                   subscribe
                                         │
                                   ┌─────┴──────┐
                                   │   Redis     │
                                   └─────┬──────┘
                                         │
                                      publish
                                         │
                                  ┌──────┴─────┐
                                  │   poller   │
                                  │ (Binance   │
                                  │ WebSocket) │
                                  └────────────┘
```

### Monorepo Structure

```
exness/
├── apps/
│   ├── web/        # Next.js 16 frontend (port 3000)
│   ├── server/     # Express REST API (port 4000)
│   ├── ws/         # WebSocket broadcast server (port 8080)
│   └── poller/     # Binance stream → Redis publisher
├── packages/
│   ├── ui/
│   ├── eslint-config/
│   └── typescript-config/
└── .github/
    └── workflows/
        └── deploy.yml   # CI/CD: build + deploy to EC2
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 16, React 19, Tailwind CSS 4, Zustand, lightweight-charts |
| Backend API | Express 5, TypeScript, JWT, bcrypt |
| Real-time | WebSocket (ws), Redis pub/sub |
| Data Source | Binance WebSocket stream (BTC/USDT) |
| Package Manager | pnpm 9 (monorepo) |
| Process Manager | PM2 |
| CI/CD | GitHub Actions |
| Hosting | AWS EC2 (Ubuntu 26.04, ap-south-1) |

---

## Getting Started Locally

### Prerequisites

- Node.js >= 18
- pnpm 9
- Redis running on `localhost:6379`

### 1. Clone the repo

```bash
git clone https://github.com/jeetupal31/exness.git
cd exness
```

### 2. Install dependencies

```bash
pnpm install
```

### 3. Start Redis

```bash
# macOS
brew services start redis

# Ubuntu/Debian
sudo service redis-server start

# Windows (if Redis is installed)
redis-server
```

### 4. Run all services in development

```bash
# Terminal 1 — API server (port 4000)
cd apps/server && npm run dev

# Terminal 2 — WebSocket server (port 8080)
cd apps/ws && npm run dev

# Terminal 3 — Binance poller
cd apps/poller && npm run dev

# Terminal 4 — Next.js frontend (port 3000)
cd apps/web && npm run dev
```

Then open [http://localhost:3000](http://localhost:3000).

### 5. Build for production

```bash
# Build all services
cd apps/server  && npm run build
cd apps/ws      && npm run build
cd apps/poller  && npm run build
cd apps/web     && npm run build

# Start with PM2
pm2 start apps/server/dist/index.js  --name api-server
pm2 start apps/ws/dist/index.js      --name ws-server
pm2 start apps/poller/dist/index.js  --name poller
pm2 start "npm start" --name web     --cwd apps/web
```

---

## API Reference

### Auth

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/user/signup` | Create new account |
| `POST` | `/api/v1/user/signin` | Sign in, returns JWT |

### Orders (JWT required)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/order/open` | Open a new position |
| `GET`  | `/api/v1/order/getOpenOrder` | Get open positions |
| `POST` | `/api/v1/buy/close-order/:id` | Close a position |

### Candles

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/candles` | Historical candle data |

---

## CI/CD

Every push to `main`:

1. **Build job** — installs dependencies, compiles all 4 TypeScript/Next.js services
2. **Deploy job** — SSHes into the EC2 instance, runs `git reset --hard`, rebuilds, and restarts PM2

Pull requests to `main` run the build job only (no deploy).

### Required GitHub Secrets

| Secret | Description |
|--------|-------------|
| `EC2_HOST` | EC2 public IP address |
| `EC2_USERNAME` | SSH user (`ubuntu`) |
| `EC2_SSH_KEY` | Private key (PEM file contents) |

---

## Features

- Live BTC/USDT price feed from Binance (sub-second updates)
- Real-time candlestick chart powered by `lightweight-charts`
- Market Buy / Market Sell with configurable leverage (1x–20x)
- Stop Loss & Take Profit on orders
- Open positions table with live unrealised PnL
- JWT-based authentication (signup / signin)
- In-memory order book and position management

---

## Author

**JeetuPalhub** — [github.com/jeetupal31](https://github.com/jeetupal31)
