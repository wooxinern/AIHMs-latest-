# AIHMs Guardian

**AIHMs Guardian** (Artificial Intelligence Health Monitoring System) is a geriatric pre-triage, health monitoring, and caregiver coordination web application designed for the Malaysian healthcare ecosystem. It combines AI-assisted symptom triage, vital signs tracking, wearable integration, real-time caregiver communication, and community wellness features in a multilingual interface.

---

## Key Features

- **Role-Based Experience (`patient`, `caregiver`, `admin`)**
  - **Patients:** Log daily vitals (blood pressure, heart rate, weight, steps, sleep, mood), chat with the AI health assistant, manage appointments, connect wearables, and engage with the community.
  - **Caregivers:** Switch between linked patients, view clinical health insights and trends, manage patient appointments/wearables, and communicate via direct messages or voice/video calls.
  - **Admins:** Monitor user metrics, manage user roles and accounts, review global appointments, and triage user feedback.
- **Luminescent Guardian AI & HEATS Pre-Triage (`@google/genai`)**
  - Conversational geriatric assistant powered by `gemini-3-flash-preview` with streaming responses, Malaysian cultural context, and multilingual support.
  - Extracts structured vitals aligned with HL7 FHIR `Observation` structures and assigns **HEATS Triage Zones** (`Red`, `Yellow`, `Green`, `None`) and action triggers (`Emergency`, `Scheduling`, `Community`).
  - Generates role-tailored health summaries, trend analyses, proactive alerts, and actionable recommendations.
- **Vitals Dashboard & Health History**
  - Manual vital entry, wearable sync display, triage status cards, and historical log filtering, editing, and deletion backed by Firestore.
- **Calendar & Malaysian Healthcare Locator**
  - Month/week/day appointment tracking with browser notification reminders.
  - Interactive Leaflet map (`react-leaflet`) showing nearby Malaysian government and private hospitals and clinics (e.g., HKL, UMMC, IJN, Sunway Medical Centre, Klinik Kesihatan) sorted by distance using browser geolocation.
- **Health Circle & Real-Time Communication**
  - Link patients and caregivers via invite tokens and manage trusted emergency contacts.
  - Real-time peer-to-peer chat and WebRTC voice/video calling powered by **Socket.io**.
- **Community & Wellness Circles**
  - Public wellness feed for events, health tips, and milestones with likes, comments, bookmarks, RSVPs, and interest circles.
- **Wearables Integration**
  - Fitbit OAuth 2.0 flow and data sync (steps, resting heart rate, sleep hours) via Express backend endpoints, plus a simulated smartwatch sync mode for testing.
- **Multilingual & Voice Accessibility**
  - Supports **Bahasa Malaysia (`ms`)**, **English (`en`)**, and **Mandarin Chinese (`zh`)**.
  - Built-in Web Speech API voice recognition and speech synthesis narration.

---

## Tech Stack

- **Frontend:** React 19, TypeScript, Vite, Tailwind CSS v4, Motion (`motion/react`), Lucide Icons, Recharts, React Markdown, Leaflet (`react-leaflet`)
- **Backend:** Express (`server.ts`), Socket.io, Axios, Cookie Parser, Dotenv
- **AI:** Google GenAI SDK (`@google/genai`) using `gemini-3-flash-preview`
- **Database & Auth:** Firebase Authentication & Cloud Firestore (configured via `firebase-applet-config.json` and `firestore.rules`)

---

## Project Structure

```text
├── server.ts                       # Express server, Vite dev middleware, Socket.io signaling, Fitbit OAuth APIs
├── firebase-applet-config.json     # Firebase client configuration
├── firebase-blueprint.json         # Firestore entity schemas and collection map
├── firestore.rules                 # Firestore security rules
├── metadata.json                   # App metadata and browser permissions
├── .env.example                    # Environment variable template
└── src/
    ├── App.tsx                     # Root routing, auth gate, layout shell
    ├── firebase.ts                 # Firebase initialization and centralized handleFirestoreError utility
    ├── types.ts                    # Shared TypeScript interfaces (UserProfile, HealthData, FHIRVitalSigns, etc.)
    ├── components/
    │   ├── FirebaseProvider.tsx    # Auth state, user profile sync, role/profile helpers
    │   ├── LanguageProvider.tsx    # i18n translations (ms, en, zh)
    │   ├── ThemeProvider.tsx       # Theme management
    │   ├── SocketProvider.tsx      # Socket.io client connection
    │   ├── OnboardingTutorial.tsx  # Guided onboarding flow
    │   ├── auth/AuthForms.tsx      # Login and registration forms
    │   ├── communication/CallManager.tsx # WebRTC voice/video call context & UI
    │   └── layout/                 # TopAppBar, BottomNavBar, SideDrawer
    ├── screens/
    │   ├── Dashboard.tsx           # Patient/caregiver vitals overview and manual logging
    │   ├── Insights.tsx            # AI clinical summaries, Recharts trends, recommendations
    │   ├── Chat.tsx                # AI health companion chat interface
    │   ├── HealthHistory.tsx       # Historical vitals archive (view, edit, delete)
    │   ├── Calendar.tsx            # Appointments and Leaflet hospital/clinic map
    │   ├── HealthCircle.tsx        # Caregiver linkage, trusted contacts, messaging & calls
    │   ├── Community.tsx           # Community posts, events, and circles
    │   ├── Wearables.tsx           # Fitbit OAuth and simulated wearable sync
    │   ├── Settings.tsx            # Profile, role switching, privacy, language, narration
    │   └── Admin.tsx               # User management, appointments overview, feedback triage
    └── services/
        ├── geminiService.ts        # AI chat streaming, HEATS triage, and insights generation
        ├── notificationService.ts  # Browser notification permissions and appointment alerts
        └── voiceService.ts         # Browser SpeechSynthesis and SpeechRecognition helpers
```

---

## Environment Variables

Copy `.env.example` to `.env` and populate the required values:

```env
# Fitbit OAuth Credentials (optional, required for live Fitbit sync)
FITBIT_CLIENT_ID=
FITBIT_CLIENT_SECRET=

# AI Configuration (required for Gemini chat & insights)
GEMINI_API_KEY=

# App Configuration
APP_URL=http://localhost:3000
```

> **Note:** Firebase project settings are read from `firebase-applet-config.json`.

---

## Getting Started

### Prerequisites

- Node.js 20+ (or compatible runtime)
- npm

### Installation & Development

1. **Install dependencies:**
   ```bash
   npm install
   ```

2. **Start the development server (Express + Socket.io + Vite middleware on port 3000):**
   ```bash
   npm run dev
   ```

3. **Type-check:**
   ```bash
   npm run lint
   ```

4. **Build for production:**
   ```bash
   npm run build
   ```

---

## API Endpoints (`server.ts`)

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/api/auth/fitbit/url` | Generates the Fitbit OAuth 2.0 authorization URL for a user (`?uid=...`). |
| `GET` | `/auth/fitbit/callback` | Handles the Fitbit OAuth callback, exchanges the auth code for tokens, and notifies the opener window. |
| `GET` | `/api/fitbit/sync` | Fetches today's steps, resting heart rate, and sleep duration from the Fitbit API (`?accessToken=...`). |

Real-time chat and WebRTC signaling (`register-user`, `join-room`, `send-message`, `typing`, `call-user`, `answer-call`, `ice-candidate`, `end-call`) run over Socket.io on the same HTTP server.

---

## Firestore Data Model

Core collections defined in `firebase-blueprint.json` and secured via `firestore.rules`:

- `/users/{userId}` — User profiles, roles, caregiver/patient links, latest vitals, and wearable status
- `/users/{userId}/healthLogs/{logId}` — Health vitals entries and AI triage analysis
- `/users/{userId}/appointments/{apptId}` — Scheduled medical and community appointments
- `/users/{userId}/trustedContacts/{contactId}` — Emergency and family contacts
- `/users/{userId}/chatHistory/{messageId}` — AI assistant conversation history
- `/conversations/{conversationId}/messages/{messageId}` — Direct messages between patients and caregivers
- `/communityPosts/{postId}` & `/circles/{circleId}` — Community feed posts, comments, and wellness circles
- `/feedback/{feedbackId}` — User bug reports and feature suggestions
