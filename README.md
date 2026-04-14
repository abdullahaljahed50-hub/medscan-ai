# 🏥 medscan-ai

> A Telegram-based AI agent that scans handwritten doctor prescriptions and delivers clear, patient-friendly medication summaries — powered by GPT-4o-mini vision and Google Gemini.

<p align="center">
  <img src="https://img.shields.io/badge/Built%20With-n8n-orange?style=for-the-badge&logo=n8n" />
  <img src="https://img.shields.io/badge/AI-GPT--4o--mini-412991?style=for-the-badge&logo=openai" />
  <img src="https://img.shields.io/badge/LLM-Google%20Gemini-4285F4?style=for-the-badge&logo=google" />
  <img src="https://img.shields.io/badge/Platform-Telegram-2CA5E0?style=for-the-badge&logo=telegram" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" />
</p>

---

## 📌 Overview

Handwritten prescriptions are notoriously difficult to read — even for trained professionals. **medscan-ai** solves this by letting any patient simply photograph their prescription and send it to a Telegram bot.

Within seconds, the bot responds with a structured, plain-language breakdown of every medication, dosage, frequency, warning, and next step — flagging anything illegible for pharmacist verification.

For follow-up questions, the bot remembers the conversation context, enabling a natural back-and-forth without losing track of the prescription details.

---

## 🖼️ Workflow Diagram

![medscan-ai Workflow](Doctor_Prescription_Analyzer_Agent.png)

---

## ✨ Features

| Feature | Description |
|---|---|
| 📷 Prescription Scan | Send a photo of any handwritten prescription for instant AI analysis |
| 💬 Conversational Follow-up | Ask plain-English questions about medications, doses, or side effects |
| 🧠 Session Memory | Retains the last 10 messages per user for coherent multi-turn conversations |
| ⚠️ Safety Flagging | Illegible items and high-risk/controlled substances are explicitly flagged for verification |
| 🤖 Dual-Model Pipeline | GPT-4o-mini handles vision/OCR; Google Gemini drives the conversational agent |
| 📲 Zero Friction UX | Runs entirely in Telegram — no app download or signup needed for end users |

---

## 🏗️ Architecture

```
Telegram Trigger
      │
      ▼
   Switch (Rules)
   ├── [Text Message]  ──────────────────────────────────────┐
   │                                                          │
   └── [Photo] → Analyze Image (GPT-4o-mini) → Ai Response ──┤
                                                              │
                                                              ▼
                                                   AI Agent (Google Gemini)
                                                   ├── Chat Model : Gemini
                                                   └── Memory    : Buffer Window (per Chat ID)
                                                              │
                                                              ▼
                                                   Send Text Message (Telegram)
```

### Node Reference

| Node | Type | Role |
|---|---|---|
| **Telegram Trigger** | Trigger | Receives incoming messages — both text and photos |
| **Switch** | Router | Routes to the text path or image path based on message type |
| **Analyze Image** | OpenAI GPT-4o-mini | Performs OCR and structured clinical analysis on the prescription image |
| **Ai Response** | Set | Extracts and normalizes the GPT analysis output |
| **User Query** | Set | Captures the raw text message from the user |
| **AI Agent** | LangChain Agent | Orchestrates the final response using memory and the LLM |
| **Google Gemini Chat Model** | LLM | Powers conversational understanding and response generation |
| **Simple Memory** | Buffer Window | Stores per-user session context keyed by Telegram chat ID |
| **Send a text message** | Telegram | Delivers the final response back to the patient |

---

## 🧠 Medical Scribe Prompt Logic

The image analysis node instructs GPT-4o-mini to behave as a **virtual medical scribe with 50+ years of clinical experience**. Every prescription scan produces output in five fixed sections:

```
1. 📋 Readable Summary
   One-to-three sentence overview of the prescription's apparent purpose

2. 💊 Medications
   Per drug: Name · Strength · Form · Dose · Route · Frequency · Duration · Special Instructions

3. ⚠️ Important Instructions / Warnings
   Non-medication instructions, follow-up timing, red flags to watch for

4. 🔍 Unclear / Illegible Items
   Any unreadable word, dose, or instruction explicitly marked as UNREADABLE — VERIFY

5. ✅ Next Steps for Patient
   One or two clear action items (verify unclear items, bring original to pharmacy, etc.)
```

> **Safety rule:** medscan-ai never suggests alternative medications, new diagnoses, or modified doses. Confidence below 90% on any item is always flagged — never guessed.

---

## 🚀 Getting Started

### Prerequisites

- [n8n](https://n8n.io/) — self-hosted or cloud (v1.0+)
- A Telegram Bot Token — [create via BotFather](https://t.me/BotFather)
- An OpenAI API Key — [platform.openai.com](https://platform.openai.com/api-keys)
- A Google Gemini API Key — [Google AI Studio](https://aistudio.google.com/app/apikey)

### Installation

**1. Clone the repository**
```bash
git clone https://github.com/YOUR_USERNAME/medscan-ai.git
cd medscan-ai
```

**2. Import the workflow into n8n**
- Open your n8n dashboard
- Navigate to **Workflows → Import from File**
- Select `Doctor_Prescription_Analyzer_Agent.json`

**3. Add your credentials**

In n8n, go to **Credentials** and create the following:

| Credential Name | Type | Where to Get |
|---|---|---|
| Telegram account | Telegram API | [BotFather](https://t.me/BotFather) → `/newbot` |
| OpenAI account | OpenAI API | [platform.openai.com](https://platform.openai.com/api-keys) |
| Google Gemini (PaLM) Api account | Google PaLM API | [Google AI Studio](https://aistudio.google.com/app/apikey) |

**4. Activate & expose your n8n instance**
- Toggle the workflow to **Active**
- Your n8n instance must be publicly reachable for Telegram webhooks to work
- For local development, use a tunnel such as [ngrok](https://ngrok.com/) or [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)

---

## 💬 Usage

### Scanning a Prescription
Send any **photo** of a handwritten prescription to the Telegram bot.

### Asking Follow-Up Questions
Send a **text message** at any time — e.g.:
- *"What are the side effects of Amoxicillin?"*
- *"Can I take this medication with food?"*
- *"What does this dosage mean?"*

### Sample Bot Output

```
📋 Readable Summary
The prescription appears to treat a bacterial respiratory infection.
Antibiotics and a cough suppressant have been prescribed for a short course.

💊 Medications
• Amoxicillin 500mg — Capsule — 1 capsule — oral — 3× daily — 7 days — take with food
• Dextromethorphan 15mg — Syrup — 10ml — oral — every 6 hours — 5 days

⚠️ Important Instructions / Warnings
• Complete the full antibiotic course even if you feel better
• Return for follow-up in 7 days if symptoms do not improve

🔍 Unclear / Illegible Items
• Third medication name: UNREADABLE — VERIFY with your doctor or pharmacist

✅ Next Steps for Patient
1. Confirm the unreadable item with your prescribing doctor before purchase
2. Bring the original prescription to the pharmacy
```

---

## 🛠️ Customization

**Upgrade the vision model**
In the *Analyze Image* node, change `gpt-4o-mini` to `gpt-4o` for significantly better accuracy on difficult or faded handwriting.

**Adjust conversation memory**
In the *Simple Memory* node, change `contextWindowLength` (default: `10`) to retain more or fewer messages per session.

**Add drug interaction lookup**
Connect a tool node (e.g., an HTTP request to a drug database API) to the AI Agent to enable real-time interaction checking.

**Multi-language output**
Add a language instruction to the system prompt in the *Analyze Image* node to receive summaries in Bengali, Arabic, Spanish, or any other language.

---

## 🔒 Privacy & Safety Disclaimer

- **medscan-ai is a reading aid only** — it does not provide medical advice, diagnoses, or treatment recommendations
- Prescription images are processed transiently and are not stored by this workflow
- Always verify the bot's output with a licensed pharmacist or physician before taking any medication
- This tool is **not a substitute** for professional medical consultation

---

## 📁 Repository Structure

```
medscan-ai/
├── Doctor_Prescription_Analyzer_Agent.json   # n8n workflow (importable)
├── Doctor_Prescription_Analyzer_Agent.png    # Workflow architecture diagram
└── README.md                                 # This file
```

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m 'Add my feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a Pull Request

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

## 🙏 Acknowledgements

- [n8n](https://n8n.io/) — Open-source workflow automation platform
- [OpenAI](https://openai.com/) — GPT-4o-mini multimodal vision model
- [Google Gemini](https://deepmind.google/technologies/gemini/) — Conversational AI backbone
- [Telegram Bot API](https://core.telegram.org/bots/api) — Real-time messaging interface
