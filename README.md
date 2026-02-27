# XUI-Manager

A production-oriented Telegram bot and payment backend for managing **XUI/Xray VPN services**.

XUI-Manager combines:
- User onboarding and service purchase flows in Telegram.
- Server/inbound/client automation via the XUI panel API.
- Wallet, ranking, ticketing, and service lifecycle tools.
- Payment integrations (ZarinPal and Cryptomus).
- A FastAPI callback service for payment verification and finalization.

---

## Table of Contents

- [Overview](#overview)
- [Core Features](#core-features)
- [Project Structure](#project-structure)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the Project](#running-the-project)
- [Database](#database)
- [Payment Flow](#payment-flow)
- [Operational Notes](#operational-notes)
- [Security Best Practices](#security-best-practices)
- [Troubleshooting](#troubleshooting)

---

## Overview

This repository provides a complete management layer around XUI/Xray services, centered on Telegram interactions.
It is designed for teams that sell and manage VPN subscriptions and need:

- Automated provisioning and renewal,
- Multiple payment methods,
- Administrative moderation tools,
- Service usage and support workflows.

---

## Core Features

### User-facing capabilities
- Telegram bot onboarding and account setup.
- Service browsing and purchase.
- Service upgrade and personalization.
- Wallet top-up and wallet-based payments.
- Ticketing/support flow.
- Ranking/reward mechanics.
- Daily gift and notification workflows.

### Admin capabilities
- Add/update inbounds and services.
- Broadcast messages to all users.
- Add credit to user wallets.
- Inspect and moderate user requests.
- Service cleanup and maintenance operations.

### Infrastructure and automation
- XUI API integration for inbound/client management.
- Automatic connection refresh/session handling.
- Payment callback handling with FastAPI templates.
- SQLite persistence for users, services, purchases, tickets, and payment records.

---

## Project Structure

```text
.
├── main.py                      # Telegram bot entry point and handlers
├── bot_start.py                 # Start/menu and initial bot flows
├── tasks.py                     # Main user task handlers and business logic
├── admin_task.py                # Admin command/task handlers
├── api_clean.py                 # XUI panel API wrapper and operations
├── database.py                  # SQLite schema initialization
├── sqlite_manager.py            # Database helper/manager utilities
├── private.py                   # Runtime configuration (tokens, domains, pricing)
├── zarinPalFastAPI.py           # FastAPI callback for ZarinPal payment results
├── zarinPalFastAPIUtilities.py  # Helper methods for ZarinPal callback actions
├── cryptomus/                   # Cryptomus payment integration
├── zarinPal/                    # ZarinPal bot integration
├── templates/                   # FastAPI HTML templates (success/fail pages)
└── req.txt                      # Python dependencies
```

---

## Architecture

1. **Telegram bot layer** (`main.py`) registers command handlers and callback routes.
2. **Business logic layer** (`tasks.py`, `admin_task.py`, `bot_start.py`) handles user/admin workflows.
3. **XUI integration layer** (`api_clean.py`) communicates with remote XUI panels.
4. **Persistence layer** (`database.py`, `sqlite_manager.py`) stores operational data in SQLite.
5. **Payment callback layer** (`zarinPalFastAPI.py`) verifies and finalizes payments.

---

## Tech Stack

- **Python**
- **python-telegram-bot==13.15**
- **FastAPI + Uvicorn**
- **aiohttp / requests**
- **SQLite**
- **Jinja2 templates**

Dependencies are listed in `req.txt`.

---

## Requirements

- Python 3.10+
- Linux server (recommended for deployment)
- Network access from your server to:
  - Telegram Bot API
  - XUI panel endpoints
  - ZarinPal/Cryptomus APIs

---

## Installation

```bash
git clone <your-repo-url>
cd XUI-Manager

python3 -m venv .venv
source .venv/bin/activate

pip install -r req.txt
```

---

## Configuration

The project currently reads runtime settings from `private.py`.

Key values include:
- Telegram bot token and admin chat IDs
- XUI auth credentials and panel port
- Main domain/protocol
- Service pricing constants
- Payment provider credentials

### Recommended approach

For production, move sensitive values to environment variables and avoid committing secrets in plain text.

A practical pattern:
- Keep `private.py` as a local-only adapter that reads from `os.environ`.
- Store secrets in your process manager or `.env` (excluded from Git).

---

## Running the Project

### 1) Start Telegram bot

```bash
python main.py
```

### 2) Start payment callback API (ZarinPal)

```bash
uvicorn zarinPalFastAPI:app --host 0.0.0.0 --port 8000
```

The callback route is:

```text
GET /recive_payment_result/
```

> Note: The route name is intentionally spelled `recive` to match the current implementation.

---

## Database

Database schema is initialized in `database.py` and includes tables such as:
- `User`
- `Product`
- `Purchased`
- `Ticket`
- `Credit_History`
- `Rank`
- `Hourly_service`
- `Statistics`
- `Gift_service`
- `Cryptomus`
- `iraIranPaymentGeway`

By default, startup creates/uses:

- `v2ray.db`

---

## Payment Flow

### ZarinPal
1. User initiates payment from Telegram flow.
2. Payment metadata is stored in SQLite.
3. User is redirected to gateway.
4. Gateway returns to FastAPI callback.
5. Callback verifies payment and:
   - Delivers service, or
   - Upgrades service, or
   - Charges wallet.

### Cryptomus
- Integrated through modules under `cryptomus/` for crypto payment lifecycle handling.

---

## Operational Notes

- The bot contains a large number of callback handlers; keep callback patterns stable when refactoring.
- XUI sessions are maintained via a requests session and per-domain cookie map.
- Financial transaction files/log data are present in `financial_transactions/`.

---

## Security Best Practices

- Rotate all credentials and tokens before production use.
- Do not commit API keys, bot tokens, or merchant secrets.
- Restrict admin commands by `chat_id` checks.
- Place the callback API behind HTTPS (Nginx/Caddy recommended).
- Add centralized logging and alerting for failed payment finalization.
- Back up SQLite regularly, or migrate to PostgreSQL for higher scale.

---

## Troubleshooting

### Bot does not start
- Verify Python version and installed dependencies.
- Ensure token and admin IDs are valid.
- Confirm network egress to Telegram API.

### Payment is not finalized
- Verify callback URL in payment provider dashboard.
- Check FastAPI service reachability and HTTPS certificate.
- Inspect database payment records and provider response codes.

### XUI operations fail
- Check XUI auth credentials and panel port.
- Ensure remote panel is reachable from bot host.
- Confirm domain/protocol configuration consistency.

---

## Contributing

If you plan to extend this project:
- Keep handler names and callback patterns explicit.
- Separate payment/business logic from Telegram UI text.
- Add test coverage for payment and database edge cases where possible.

---

## License

No license file is currently included.
If you intend to open-source this repository, add a license (e.g., MIT, Apache-2.0) before publishing.
