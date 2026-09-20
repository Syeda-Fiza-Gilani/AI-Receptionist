# 📞 AI Voice Receptionist — n8n Workflow

A free, self-hosted AI receptionist that answers incoming phone calls, has a natural conversation with the caller, and automatically books appointments straight into Google Calendar — no manual entry, no missed calls, no answering service fees.

Built entirely with free-tier tools: **n8n + Twilio + Google Gemini + Google Calendar**.

---

<img width="1411" height="650" alt="image" src="https://github.com/user-attachments/assets/5706f544-3f89-4e41-835f-902717b908c5" />


## ✨ What it does

- ☎️ Answers incoming calls on a real phone number
- 🗣️ Holds a natural, multi-turn conversation using speech-to-text and text-to-speech
- 🧠 Uses Google Gemini to understand the caller's intent and extract booking details
- 📅 Automatically creates the appointment in Google Calendar once it has enough info
- 💸 Runs on free-tier APIs — the only real cost is Twilio's per-minute call rate (fractions of a cent per call)

---

## 🧩 How it works

```
Incoming Call (Twilio)
        │
        ▼
n8n Webhook receives call event
        │
        ▼
Manage Call Session (tracks conversation state per CallSid)
        │
        ▼
Build Gemini Prompt (conversation history + booking rules)
        │
        ▼
Call Gemini API (extracts name, service, date/time)
        │
        ▼
Parse Gemini Response
        │
        ▼
   Booking complete? ──No──▶ Ask next question (TwiML) ──▶ back to caller
        │
       Yes
        │
        ▼
Create Google Calendar Appointment
        │
        ▼
Confirm booking out loud → Hang up
```

Each caller turn is a fresh HTTP request from Twilio (phone calls are stateless at the HTTP level), so the workflow keeps track of the conversation using the call's unique `CallSid` as a session key.

---

## 🛠️ Tech stack

| Component | Tool | Cost |
|---|---|---|
| Automation / orchestration | [n8n](https://n8n.io) | Free (self-hosted) |
| Phone number & call handling | [Twilio](https://twilio.com) | Free trial credit, then ~$1/mo + ~1–2¢/min |
| Speech-to-text & text-to-speech | Twilio `<Gather>` / `<Say>` | Included in Twilio's call cost |
| Conversational AI | [Google Gemini](https://aistudio.google.com) | Free tier |
| Appointment booking | [Google Calendar API](https://console.cloud.google.com) | Free |

---

## 🚀 Setup

### 1. Get your n8n webhook URL
Import `AI-Voice-Receptionist.json` into n8n, activate the workflow, and copy the production webhook URL from the **Twilio Voice Webhook** node.

### 2. Update the callback URL
Open the **Build Question TwiML** node and replace the placeholder:
```
https://YOUR-N8N-DOMAIN/webhook/voice-receptionist
```
with your real webhook URL from step 1.

### 3. Configure Twilio
- Buy a Twilio number with Voice capability.
- In the Twilio Console → your number → **"A Call Comes In"** → set to **Webhook**, paste your n8n webhook URL, method `POST`.

### 4. Add credentials in n8n
- **Google Gemini** — API key from [Google AI Studio](https://aistudio.google.com/app/apikey)
- **Google Calendar** — OAuth2 credentials from [Google Cloud Console](https://console.cloud.google.com) (enable Calendar API first)

### 5. Test it
Call your Twilio number, have a conversation, and confirm the appointment lands on your Google Calendar.

---

## ⚠️ Known limitations

- **Session storage**: uses n8n's workflow static data keyed by `CallSid`. Fine for testing and low call volume, but not safely concurrent at scale. Swap in a real database (or even a Google Sheet row per call) for production use.
- **Single calendar / single service**: this version books to one calendar and doesn't check for conflicting time slots — add availability-checking logic before going live for a real business.
- **No call recording or transcript storage** by default — add a logging step if you need call history.

---

## 🗺️ Roadmap ideas

- [ ] Conflict-checking against existing calendar events before confirming a slot
- [ ] SMS confirmation sent to the caller after booking
- [ ] Multi-service / multi-calendar routing (e.g. different staff members)
- [ ] Persistent session storage (Postgres/Redis instead of static data)
- [ ] Call transcript logging to Google Sheets or a database

---

## 📄 License

MIT 

---

## 🙌 Credit

Built with [n8n](https://n8n.io), [Twilio](https://twilio.com), and [Google Gemini](https://ai.google.dev).
