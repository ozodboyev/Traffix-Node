# Traffix Node - Decentralized VPN & Proxy Network

![Traffix Node Banner](https://via.placeholder.com/1200x400?text=Traffix+Node+DePIN)

**Traffix Node** is a **DePIN (Decentralized Physical Infrastructure Network)** that transforms everyday Android smartphones into high-performance residential proxy nodes. By sharing unused bandwidth, users earn rewards while contributing to a censorship-resistant, global internet access layer.

This repository contains the source code for the **Traffix Node Backend**, a high-performance Go server that orchestrates the network, handles billing, and manages secure tunnels.

## 🚀 Features

*   **Decentralized Architecture:** No central exit nodes. Traffic is routed through thousands of residential mobile devices.
*   **Mobile-First DePIN:** Native Android app acting as a node (miner).
*   **Telegram Integration:** Seamless onboarding and authentication via Telegram (900M+ user base).
*   **High Performance:** Backend written in **Go (Golang)**, optimized for 10k+ concurrent tunnels per instance.
*   **Obfuscated Protocol:** Custom transport protocol to bypass DPI (Deep Packet Inspection) and censorship.
*   **Crypto Payouts:** Automated earnings settlement in **USDT** (via NOWPayments/Blockchain integration).
*   **Smart Routing:** AI-driven session management to ensure optimal speed and IP quality.

## 🛠 Tech Stack

*   **Language:** Go (Golang) 1.24+
*   **Database:** PostgreSQL (TimescaleDB for metrics)
*   **Caching:** Redis (Session management & Rate limiting)
*   **Proxy Protocol:** SOCKS5 / HTTP (Custom Tunneling)
*   **Auth:** Telegram Login Widget & JWT
*   **Infrastructure:** Linux (Systemd), Nginx (Reverse Proxy)

## 📦 Installation

### Prerequisites

*   Go 1.22 or higher
*   PostgreSQL 14+
*   Redis 6+
*   Git

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/traffix-node.git
cd traffix-node/backend
```

### 2. Configure Environment

Copy the example config and edit it with your credentials:

```bash
cp config.example.yaml config.yaml
nano config.yaml
```

### 3. Build and Run

```bash
go mod download
go build -o traffix-server cmd/server/main.go
./traffix-server
```

## 📱 Mobile App (Node)

The Android application source code is located in the `android/` directory. It is built with **Kotlin** and **Jetpack Compose**.

## 🤝 Contributing

We welcome contributions from the community! Whether it's fixing a bug, improving documentation, or proposing new features. Please read `CONTRIBUTING.md` for details.

## 📄 License

This project is licensed under the [MIT License](LICENSE).

## 🔗 Links

*   **Website:** [Your Website]
*   **Telegram Bot:** [@YourBot]
*   **Grant Proposal:** [Peaq Network / TON Foundation]
