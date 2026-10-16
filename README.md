<div align="center">

# WhatsApp AI Assistant

**An interactive WhatsApp bot built on the Meta Cloud API, using OpenAI to generate conversational responses — the conversational front-end for [Velo.Stor](https://velo-stor.netlify.app/).**

[**🔗 Companion project — Velo.Stor**](https://github.com/El-Tousy/velo.Store) · [**📄 Deutsch**](README.de.md)

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-000000?style=flat&logo=flask&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI_API-412991?style=flat&logo=openai&logoColor=white)
![Meta](https://img.shields.io/badge/Meta_Cloud_API-0866FF?style=flat&logo=meta&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [API Endpoints](#api-endpoints)
- [Testing](#testing)
- [Deployment](#deployment)
- [Technical Challenges and Learnings](#technical-challenges-and-learnings)
- [Roadmap](#roadmap)
- [Related Project](#related-project)
- [Author](#author)
- [License](#license)

---

## Overview

This project is a webhook server that turns WhatsApp into a conversational storefront. It receives customer messages through the **Meta WhatsApp Cloud API**, interprets them with the **OpenAI API**, and replies automatically — using the same product catalogue as the [Velo.Stor](https://github.com/El-Tousy/velo.Store) website.

The goal is to let a customer ask about bikes, e-bikes or scooters directly in a WhatsApp thread — no app install, no website visit required — which matches how small businesses actually sell in markets where WhatsApp is the primary commerce channel.

> **Note on scope.** This is an internship / learning project. The bot and the catalogue it draws from use sample data; it is not a production commercial deployment.

---

## Features

- **Webhook verification** — handles Meta's `GET /webhook` handshake (`hub.challenge` / `hub.verify_token`)
- **Inbound message handling** — receives and parses `POST /webhook` payloads from the Meta Cloud API
- **AI-generated replies** — customer messages are sent to the OpenAI API, which generates a natural-language response grounded in the product catalogue
- **Shared catalogue** — answers are based on the same product data as the Velo.Stor website, so the two front-ends stay consistent
- **Signature / token verification** — validates incoming requests before processing them
- **Structured logging** — request and error logs for debugging webhook delivery issues

---

## Architecture

```mermaid
flowchart LR
    A[Customer] -->|WhatsApp message| B[Meta Cloud API]
    B -->|POST /webhook| C["Flask server<br/>Python"]
    C --> D{Verify request}
    D -->|valid| E[Message handler]
    D -->|invalid| Z[Reject]
    E --> F[OpenAI API]
    E --> G[(Product catalogue)]
    F --> E
    G --> E
    E -->|reply| B
    B -->|WhatsApp message| A
```

Meta forwards every inbound WhatsApp message to this server's webhook. The server verifies the request, builds a prompt that combines the customer's message with relevant catalogue data, sends it to the OpenAI API, and returns the generated reply to the customer through the Meta Cloud API — the same catalogue the [Velo.Stor](https://github.com/El-Tousy/velo.Store) website displays.

---

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Language | Python 3.11+ | Readable, strong ecosystem for API work |
| Web framework | Flask | Lightweight webhook server, no unnecessary overhead |
| Messaging | Meta WhatsApp Cloud API | Official, no third-party WhatsApp automation risk |
| AI layer | OpenAI API | Natural-language understanding and reply generation |
| Testing | pytest | Simple, readable unit tests |
| Versioning | Git & GitHub | Source control and collaboration |

---

## Project Structure

```
whatsapp-bot/
│
├── webhook.py                 # Flask server: webhook verification and message handling
├── utils.py                   # Helper functions
├── update_excel_realtime.py   # Script that updates the product data in real time
├── test_utils.py              # Unit tests
├── requirements.txt           # Python dependencies
├── screenshots/               # Screenshots of the bot in action
├── LICENSE
├── README.md
└── README.de.md
```

---

## Getting Started

### Prerequisites

- Python 3.11 or later
- A [Meta Developer](https://developers.facebook.com/) account with a WhatsApp Business app configured
- An [OpenAI API](https://platform.openai.com/) key
- `pip` and, optionally, a virtual environment tool (`venv`, `poetry`, etc.)

### Installation

```bash
git clone https://github.com/El-Tousy/whatsapp-bot.git
cd whatsapp-bot

python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

### Configuration

Create a `.env` file in the project root and fill in the variables described in [Environment Variables](#environment-variables).

### Running locally

```bash
python webhook.py
```

To receive real WhatsApp messages during local development, expose your local server with a tunnel (e.g. `ngrok http 5000`) and register the resulting HTTPS URL as your webhook callback URL in the Meta Developer dashboard.

---

## Environment Variables

| Variable | Description |
|---|---|
| `WHATSAPP_TOKEN` | Access token for the Meta Cloud API (system user token) |
| `PHONE_NUMBER_ID` | ID of the WhatsApp Business phone number sending/receiving messages |
| `VERIFY_TOKEN` | Arbitrary string you choose; must match the value entered in the Meta webhook configuration |
| `APP_SECRET` | App secret used to verify Meta's request signature |
| `OPENAI_API_KEY` | API key for the OpenAI API |
| `PORT` | Port the Flask server listens on (default `5000`) |

The `.env` file contains secrets and must never be committed to the repository.

---

## API Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/webhook` | One-time verification handshake used by Meta when registering the webhook |
| `POST` | `/webhook` | Receives inbound WhatsApp messages and triggers the reply flow |

---

## Testing

```bash
pytest
```

The unit tests in `test_utils.py` cover the helper functions in isolation, so they do not depend on live Meta or OpenAI credentials.

---

## Deployment

The bot needs a publicly reachable HTTPS URL so that Meta can deliver webhook events to it. Any hosting provider that can run a Python/Flask application works.

1. Deploy the application and install the dependencies from `requirements.txt`.
2. Set the environment variables listed above in the hosting provider's dashboard.
3. Register the public URL (ending in `/webhook`) in the Meta Developer dashboard, under **WhatsApp → Configuration**, using the same `VERIFY_TOKEN`.

For local development, a tunnel such as `ngrok` replaces steps 1 and 2.

---

## Technical Challenges and Learnings

- **Meta's webhook verification handshake.** The `GET /webhook` challenge/response flow, plus signature verification (`X-Hub-Signature-256`) on every `POST`, has to be implemented correctly or Meta will silently stop delivering messages — there is no verbose error returned to the developer.
- **Prompting for grounded answers.** Getting the OpenAI API to answer from the actual product catalogue, rather than inventing plausible-sounding but wrong details, took more prompt iteration than the integration code itself.
- **Sharing one source of truth with the website.** Keeping the bot's catalogue and the Velo.Stor website's catalogue in sync is the main design constraint of the whole two-repo system.
- **Latency inside the webhook window.** Calling the OpenAI API while Meta waits for a response means the handler has to stay fast and fail gracefully, otherwise Meta retries the delivery and the customer may receive duplicate replies.

---

## Roadmap

- [x] Webhook verification and message handling
- [x] OpenAI-generated replies
- [x] Shared catalogue with the Velo.Stor storefront
- [ ] CI pipeline running lint and tests on every push
- [ ] Conversation history / context per customer
- [ ] Human handoff for requests the bot can't answer
- [ ] Structured order/status lookups
- [ ] Expand test coverage

---

## Related Project

**[Velo.Stor](https://github.com/El-Tousy/velo.Store)** — the RTL Arabic e-commerce website this bot shares its catalogue with.

---

## Author

**El-Tousy** — Computer Science student, Morocco

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/El-Tousy)
[![Email](https://img.shields.io/badge/Email-D14836?style=flat&logo=gmail&logoColor=white)](mailto:leilaeltousy@gmail.com)

---

## License

Released under the MIT License. See [LICENSE](LICENSE) for details.

---

<div align="center">

⭐ If this project is useful to you, consider leaving a star.

</div>
