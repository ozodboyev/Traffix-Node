# Peaq Network Ecosystem Grant Application

## Traffix Node — Decentralized Residential Proxy Network (DePIN)

---

## 1. Project Overview

| Field | Detail |
|---|---|
| **Project Name** | Traffix Node |
| **Category** | DePIN (Decentralized Physical Infrastructure Network) |
| **Stage** | Private Beta — MVP Complete |
| **Website** | [https://traffix.uz](https://traffix.uz) |
| **GitHub** | [github.com/ozodboyev/Traffix-Node](https://github.com/ozodboyev/Traffix-Node) |
| **APK Download** | [v1.0.6 Release](https://github.com/ozodboyev/Traffix-Node/releases/tag/v1.0.6) |
| **Location** | Uzbekistan, Urgench |

---

## 2. Executive Summary

Traffix Node is a **Decentralized Physical Infrastructure Network (DePIN)** that converts everyday Android smartphones into high-performance residential proxy exit nodes. By contributing their idle internet bandwidth, node operators earn cryptocurrency rewards (USDT), while data consumers gain access to a globally distributed pool of clean, residential IP addresses.

The project is **fully built and operational in Private Beta**. We have a production-grade Golang backend, a polished native Android application, and an administrative control panel — all ready for scale. We are applying for the **Peaq Network Ecosystem Grant** to integrate Peaq's decentralized machine identity layer (peaq ID) into our node infrastructure, enabling trustless verification, on-chain reputation, and transparent reward distribution across the network.

---

## 3. The Problem

The residential proxy industry is valued at **$7.8B** and growing at **24% CAGR**, yet it is plagued by:

1. **Centralization:** A handful of companies (Bright Data, Smartproxy) control 80%+ of the market, creating single points of failure and censorship vulnerability.
2. **Opacity:** Node operators (bandwidth contributors) have zero visibility into how their traffic is used or how earnings are calculated.
3. **Exploitative Economics:** Intermediaries capture 70-90% of the revenue, leaving node operators with minimal compensation.
4. **Trust Deficit:** There is no verifiable way to prove that a node is a genuine residential device vs. a datacenter VM or emulator.

---

## 4. Our Solution: Traffix Node

Traffix Node addresses every pain point above through decentralization:

### 4.1 How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                     Traffix Node Network                     │
│                                                              │
│  ┌─────────┐    Secure Tunnel    ┌─────────────┐            │
│  │ Android  │◄──────────────────►│  Go Backend  │            │
│  │  Node    │   (WebSocket/TLS)  │ Orchestrator │            │
│  │ (Miner)  │                    │              │            │
│  └─────────┘                     └──────┬───────┘            │
│       │                                 │                    │
│       │  Shares bandwidth               │  Routes traffic    │
│       │  Earns USDT                     │  Bills consumers   │
│       ▼                                 ▼                    │
│  ┌─────────┐                    ┌──────────────┐            │
│  │  peaq    │  On-chain ID &    │   Data        │            │
│  │  Network │  Reward Settlement│   Consumers   │            │
│  └─────────┘                    └──────────────┘            │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 Key Features (All Live in MVP)

| Feature | Status | Description |
|---|---|---|
| **One-Tap Node Activation** | ✅ Live | Users press "Connect" and instantly join the network |
| **Real-Time Earnings Dashboard** | ✅ Live | Live balance, daily stats, traffic counters |
| **Billing & Transaction History** | ✅ Live | Complete ledger of all earnings, payouts, adjustments |
| **Crypto Payouts (USDT)** | ✅ Live | Automated withdrawal to BSC/ERC-20 wallets |
| **Multi-Wallet Support** | ✅ Live | Users can configure multiple payout wallets |
| **Biometric Security** | ✅ Live | Fingerprint/Face ID unlock for app security |
| **PIN Code Protection** | ✅ Live | Additional PIN layer for sensitive operations |
| **Activity Audit Log** | ✅ Live | Full history of logins, payouts, and actions |
| **Admin Control Panel** | ✅ Live | Web dashboard for network monitoring and management |
| **Telegram Auth** | ✅ Live | Seamless login via Telegram (900M+ user base) |
| **Obfuscated Protocol** | ✅ Live | Custom transport layer to bypass DPI censorship |

---

## 5. Technology Stack

### Backend (Core Orchestrator)
- **Language:** Go (Golang) 1.24 — chosen for extreme concurrency and low memory footprint
- **Database:** PostgreSQL with TimescaleDB extensions for time-series metrics
- **Cache:** Redis for real-time session state and rate limiting
- **Proxy Protocol:** Custom SOCKS5/HTTP tunneling over WebSocket with TLS 1.3
- **Architecture:** Modular monolith — single binary deployment, internally separated into independent modules (auth, billing, tunnel, checker, API gateway)

### Mobile Node (Android)
- **Language:** Kotlin with Jetpack Compose (Material 3)
- **Background Service:** Persistent foreground service with battery optimization
- **Security:** Biometric authentication, PIN protection, encrypted local storage
- **Networking:** OkHttp + Retrofit for API, custom WebSocket client for tunneling

### Admin Panel
- **Frontend:** Modern web dashboard with real-time metrics
- **Features:** User management, device tracking, IP pool monitoring, live traffic graphs, payout management, live chat, finance reporting, notification system

---

## 6. Why Peaq Network?

Peaq is the **Layer-1 blockchain purpose-built for DePIN**. It provides exactly what Traffix Node needs to evolve from a centralized-backend model to a fully decentralized, trustless network.

### 6.1 Integration Plan

| Peaq Feature | Traffix Node Application |
|---|---|
| **peaq ID** | Every Android node gets a unique on-chain machine identity. This replaces our current database-based device registration with a decentralized, tamper-proof identity system. |
| **peaq Pay** | Node operator rewards are settled on-chain via peaq pay, replacing our current off-chain USDT billing. This gives operators full transparency and eliminates trust in a central party. |
| **peaq Verify** | Before routing traffic through a node, the system verifies the node's on-chain reputation score. Nodes with high uptime and clean IPs earn higher rewards. |
| **peaq Data** | Anonymized network telemetry (uptime, bandwidth, latency) is published on-chain, creating a transparent marketplace for bandwidth. |

### 6.2 Why This Partnership Benefits Peaq

1. **Immediate Real-World DePIN Use Case:** Traffix Node is not a concept — it is a working product with a functioning payout system. Integrating with Peaq gives the ecosystem a tangible, live DePIN application.
2. **User Acquisition:** Every Android user who installs Traffix Node becomes a potential Peaq ecosystem participant. Our Telegram-based onboarding removes all Web3 friction.
3. **Network Effect:** As more nodes join, the proxy network becomes more valuable (more IPs, more locations), which attracts more consumers, which drives more rewards to node operators — a virtuous flywheel.

---

## 7. Traction & Proof of Work

### Current Metrics

> **Note:** Traffix Node is currently in **Private Alpha** and has not yet been released to the public. All metrics below reflect internal team testing only. We are intentionally holding the public launch until the Peaq integration is complete, so that users onboard directly into the decentralized version of the network.

- **Backend:** Fully operational Go server handling concurrent tunnel sessions
- **Android App:** v1.0.6 released with all core features functional
- **Admin Panel:** Complete management dashboard with real-time monitoring
- **Payout System:** 44+ completed test payouts processed through crypto rails
- **Total Traffic:** 0.37 GB routed through the network during internal alpha testing
- **Development Team:** 15 engineers across backend, mobile, infrastructure, and security

### Screenshots (Available in GitHub Repository)
- **Dashboard:** Real-time connection status, upload/download counters, daily statistics
- **Billing:** Transaction history with earnings breakdown (weekly/monthly)
- **Payout:** Wallet integration (BSC), payout request system with status tracking
- **Settings:** User profile, multi-wallet management, biometric security, audit log
- **Admin Panel:** Network-wide dashboard with user/device/traffic/payout management

---

## 8. Grant Request

### 8.1 What We Need

We are requesting a **Peaq Ecosystem Grant** to fund the following:

| Item | Purpose | Estimated Cost |
|---|---|---|
| **Peaq SDK Integration** | Implement peaq ID, peaq Pay, and peaq Verify in our Go backend and Android app | $5,000 |
| **Testnet Infrastructure** | Server costs for running incentivized testnet (3 months) | $3,000 |
| **Security Audit** | Third-party review of our custom tunneling protocol and smart contract logic | $5,000 |
| **User Incentives** | Airdrop rewards for early testnet node operators | $2,000 |
| **Total** | | **$15,000** |

### 8.2 Milestones

| Timeline | Milestone | Deliverable |
|---|---|---|
| **Month 1** | Peaq SDK Integration | peaq ID assigned to every registered device; reward calculation logic moved on-chain |
| **Month 2** | Incentivized Testnet | Public testnet launch with 500+ invited node operators; peaq Pay active for reward distribution |
| **Month 3** | Mainnet Launch | Full production deployment on Peaq mainnet; token generation event (TGE) for $TRFX governance token |

---

## 9. Team

Traffixes Node is built by a dedicated team of **15 engineers** based in Uzbekistan, covering all critical areas of the stack:

| Role | Team Size | Responsibilities |
|---|---|---|
| **Backend & Infrastructure** | 4 | Go server, tunneling protocol, database architecture, DevOps |
| **Android Development** | 4 | Native Kotlin app, background services, UI/UX (Jetpack Compose) |
| **Web & Admin Panel** | 3 | Admin dashboard, real-time monitoring, finance reporting |
| **Security & QA** | 2 | Protocol auditing, penetration testing, automated test suites |
| **Product & Operations** | 2 | Product roadmap, community management, partnerships |

**Ozodboyev Bunyod** — *Founder & Lead Architect*
- Architected the entire Traffix Node platform from the ground up
- Specializes in Go systems programming, Android native development, and distributed networking
- Based in Urgench, Uzbekistan

**GitHub:** [github.com/ozodboyev](https://github.com/ozodboyev)

---

## 10. Links & Resources

| Resource | Link |
|---|---|
| **GitHub Repository** | [github.com/ozodboyev/Traffix-Node](https://github.com/ozodboyev/Traffix-Node) |
| **APK Download (v1.0.6)** | [GitHub Release](https://github.com/ozodboyev/Traffix-Node/releases/tag/v1.0.6) |
| **Architecture Documentation** | [ARCHITECTURE.md](https://github.com/ozodboyev/Traffix-Node/blob/main/ARCHITECTURE.md) |
| **Project Roadmap** | [ROADMAP.md](https://github.com/ozodboyev/Traffix-Node/blob/main/ROADMAP.md) |
| **Website** | [traffix.uz](https://traffix.uz) |
| **Twitter (X)** | [@traffix_social](https://twitter.com/traffix_social) |
| **Discord** | [discord.gg/traffixnode](https://discord.gg/traffixnode) |
| **Telegram Community** | [@traffixcommunity](https://t.me/traffixcommunity) |

---

*Traffix Node — Decentralizing the Internet, One Phone at a Time.*
