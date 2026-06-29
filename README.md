# MailCraft AI ✉️

**AI-powered Gmail reply generator — write smarter emails in one click.**

[![Java](https://img.shields.io/badge/Java-Spring%20Boot-green?style=flat-square&logo=springboot)](https://spring.io/projects/spring-boot)
[![Gemini](https://img.shields.io/badge/AI-Google%20Gemini-blue?style=flat-square&logo=google)](https://ai.google.dev/)
[![Chrome Extension](https://img.shields.io/badge/Chrome-Extension-yellow?style=flat-square&logo=googlechrome)](https://developer.chrome.com/docs/extensions/)

---

## 🚀 Overview

**MailCraft AI** is a browser-native productivity tool that integrates directly into Gmail. It detects open compose/reply windows, injects an **"AI Reply"** button into the Gmail toolbar, and generates contextual, tone-aware email drafts using the **Google Gemini API** — all without leaving your inbox.

> **Design Decision:** The backend is intentionally stateless — no database, no authentication layer. Since this tool runs locally as a personal productivity utility, adding persistence or auth would introduce unnecessary complexity without any real benefit.

---

## 🏗️ Architecture

```
┌─────────────────────────┐        ┌──────────────────────┐
│   Gmail Web Interface   │──────► │   Chrome Extension   │
└─────────────────────────┘        └──────────┬───────────┘
                                              │
                                    POST /api/email/generate
                                    { emailContent, tone }
                                              │
                                              ▼
┌─────────────────────────┐        ┌──────────────────────┐
│     Gmail Draft Box     │◄───────│  Spring Boot Backend │
│  (AI reply auto-filled) │        └──────────┬───────────┘
└─────────────────────────┘                   │
                                              │ WebClient
                                              ▼
                                   ┌──────────────────────┐
                                   │  Google Gemini API   │
                                   └──────────────────────┘
```

**How it works:**
1. A content script runs in the background, listening for Gmail compose/reply windows to open.
2. On detection, an **"AI Reply"** button is dynamically injected into the Gmail toolbar via DOM manipulation.
3. User clicks the button → extension scrapes the active email thread from the browser DOM.
4. A `POST` request is sent to the local Spring Boot server with the scraped content and selected tone.
5. Backend constructs the prompt, calls Gemini API via **WebClient** (non-blocking HTTP), and extracts the response.
6. The generated reply is injected directly into the open Gmail compose area.

---

## ✨ Features

- 🔍 **Auto-detects** Gmail compose and reply windows via content scripts
- 💡 **Injects a native-looking "AI Reply" button** directly into the Gmail toolbar
- 🧠 **Scrapes email thread context** from the DOM for accurate, relevant replies
- 🎯 **Tone selection** — generate Professional, Casual, or Formal responses
- ⚡ **Non-blocking AI calls** — WebClient ensures the backend stays responsive
- 🪶 **Zero overhead** — no database, no auth, no friction

---

## 🛠️ Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Backend | Spring Boot (Java), Maven | Rapid REST API setup with minimal boilerplate |
| AI Integration | Google Gemini API via WebClient | Non-blocking HTTP; avoids RestTemplate's threading overhead |
| Extension | JavaScript, Chrome Extension API | Direct DOM access to Gmail's compose window |
| Build Tool | Maven (`pom.xml`) | Dependency management and build lifecycle |

---

## 📡 API Reference

### Generate Email Reply

Constructs a context-aware, tone-specific email reply using Gemini AI.

| | |
|---|---|
| **URL** | `/api/email/generate` |
| **Method** | `POST` |
| **Content-Type** | `application/json` |

**Request Body**
```json
{
  "emailContent": "Hey, are we still on for the startup sync tomorrow at 10 AM? Let me know.",
  "tone": "professional"
}
```

**Response `200 OK`**
```json
"Hi,\n\nYes, confirming that we are still on for tomorrow at 10 AM. Looking forward to it.\n\nBest regards,\n[Your Name]"
```

---

## 🚀 Getting Started

### Prerequisites

- Java 17+
- Maven
- Google Chrome
- A free [Gemini API key](https://aistudio.google.com/app/apikey)

---

### Step 1 — Configure & Run the Backend

```bash
git clone https://github.com/ShrutiShawDev/MailCraft-Ai.git
cd MailCraft-Ai/mailcraft-ai-sb
```

Open `src/main/resources/application.properties` and add:

```properties
spring.ai.gemini.api-key=YOUR_GEMINI_API_KEY
server.port=8080
```

```bash
mvn spring-boot:run
```

✅ Tomcat starts on `http://localhost:8080`

---

### Step 2 — Load the Chrome Extension

1. Open Chrome → `chrome://extensions/`
2. Toggle **Developer mode** (top-right)
3. Click **Load unpacked**
4. Select the `mailcraft-ai-ext/` folder
5. Extension is now active ✅

> After any JS changes, click the **reload** icon on the extension card.

---

### Step 3 — Test on Gmail

1. Open [Gmail](https://mail.google.com) → **Compose** or **Reply**
2. The **"AI Reply"** button appears in the formatting toolbar
3. Click it → draft is generated and auto-filled 🎉

---

## 📁 Project Structure

```
MailCraft-Ai/
├── .github/                        # GitHub Actions workflows
├── mailcraft-ai-sb/                # Spring Boot backend
│   ├── src/main/java/com/mailcraft/
│   │   ├── controller/             # EmailController — POST /api/email/generate
│   │   ├── service/                # EmailService — WebClient + Gemini prompt logic
│   │   └── dto/                    # EmailRequest / response DTOs
│   └── src/main/resources/
│       └── application.properties  # Gemini API key config
├── mailcraft-ai-ext/               # Chrome Extension
│   ├── manifest.json               # Permissions, content script registration
│   ├── content.js                  # DOM injection, scraping, backend HTTP call
│   └── popup.html                  # Extension popup UI
└── mailcraft-ai-frontend/          # React frontend
```
