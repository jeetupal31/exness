# Exness Trading Platform

[![Build & Deploy](https://github.com/jeetupal31/exness/actions/workflows/deploy.yml/badge.svg)](https://github.com/jeetupal31/exness/actions/workflows/deploy.yml)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-000?style=flat-square&logo=nextdotjs)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)
![AWS EC2](https://img.shields.io/badge/AWS_EC2-FF9900?style=flat-square&logo=amazonaws&logoColor=white)

> A production-grade real-time trading platform streaming live BTC/USDT prices from Binance with full order management, candlestick charts, and leverage-based position tracking.

## Live Demo

| Service | URL |
|---------|-----|
| Frontend | http://3.110.62.127:3000 |
| REST API | http://3.110.62.127:4000/api/v1 |
| WebSocket | ws://3.110.62.127:8080 |

## What It Does

- **Live price feed** — Binance WebSocket streams BTC/USDT ticks into Redis pub/sub in real time
- **Candlestick charts** — powered by `lightweight-charts`, renders 1m OHLC candles
- **Order management** — Market Buy / Sell with Stop Loss, Take Profit, and leverage (1x–20x)
- **Open positions table** — shows live unrealised PnL per position
- **JWT auth** — signup / signin with bcrypt-hashed passwords
- **CI/CD** — GitHub Actions builds and deploys automatically on every push to `main`

## Architecture

```
Browser (Next.js :3000)
    │
    ├── REST  ──► Express API (:4000) ──► In-memory store + Redis
    │
    └── WS    ──► WS Server  (:8080) ──► Redis SUBSCRIBE
                                              │
                                         Redis PUBLISH
                                              │
                                         Poller (Binance WebSocket)
```

## Tech Stack

| Layer | Tech |
|-------|------|
| Frontend | Next.js 16, React 19, Tailwind CSS 4, Zustand, lightweight-charts |
| Backend | Express 5, TypeScript, JWT, bcrypt |
| Real-time | WebSocket (ws), Redis pub/sub |
| Data | Binance WebSocket stream (BTC/USDT) |
| Infra | AWS EC2 (Ubuntu), PM2, GitHub Actions |
| Monorepo | Turborepo + pnpm workspaces |

## Project Structure

```
apps/
├── web/      # Next.js frontend     (port 3000)
├── server/   # Express REST API     (port 4000)
├── ws/       # WebSocket server     (port 8080)
└── poller/   # Binance → Redis feed
```

## Local Setup

```bash
# Prerequisites: Node 20+, pnpm 9, Redis running on :6379

git clone https://github.com/jeetupal31/exness.git
cd exness
pnpm install

# Run each in a separate terminal
cd apps/server  && npm run dev   # :4000
cd apps/ws      && npm run dev   # :8080
cd apps/poller  && npm run dev   # streams from Binance
cd apps/web     && npm run dev   # :3000
```

## API Endpoints

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/api/v1/user/signup` | — | Register |
| POST | `/api/v1/user/signin` | — | Login → JWT |
| POST | `/api/v1/order/open` | JWT | Open position |
| GET  | `/api/v1/order/getOpenOrder` | JWT | List open positions |
| POST | `/api/v1/buy/close-order/:id` | JWT | Close position |
| GET  | `/api/v1/candles` | — | Candle history |

## CI/CD

Push to `main` → GitHub Actions builds all 4 services → SSHes into EC2 → rebuilds → `pm2 restart all`.

---

Made with ❤️ by [Jeetu Pal](https://github.com/jeetupal31)
