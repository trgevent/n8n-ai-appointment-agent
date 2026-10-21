# AI Appointment Agent (n8n + Google Calendar)

An **n8n** workflow where an LLM agent books, reschedules and cancels appointments on Google Calendar — with real availability checks, business-hour rules, Turkish public holidays and identity verification before any change.

Built for small service businesses (barbers, beauty salons, dental clinics, small medical centers) that take appointments by phone today.

## Two entry points
| Workflow | Entry | Use it for |
|---|---|---|
| `appointment-agent-form.json` | n8n web form (name, phone, date, time) | A booking link you can share or put on a QR code |
| `appointment-agent-chat-webhook.json` | `POST /webhook/randevu` chat API | A chat widget on your website or a messaging bot |

## How it works (form version)
```
Web form (name, phone, date, time)
        │
        ▼
   AI Agent (gpt-4o-mini, memory per phone number)
        │  tools:
        ├── Check availability ─┐
        ├── Create event        │
        ├── Search events       ├── Google Calendar
        ├── Update event        │
        └── Cancel event ───────┘
        │
        ├──► Form completion page (answer to customer)
        └──► Slack notification (to the business)
```

## What the agent enforces
- **Working hours:** Mon–Fri 08:00–22:00, Sat 11:00–17:00, Sun closed; 45-min service + 15-min break, slots start on the hour
- **Public holidays:** full-day and half-day (arife) closures checked *before* anything else
- **Real availability:** checks the calendar; if the slot is taken, suggests free slots on the same day
- **Confirmation:** asks once before writing to the calendar
- **Security:** never reveals, changes or cancels a booking without an exact **name + phone** match
- **Language:** replies in the customer's language

## Chat API (webhook version)
```bash
curl -X POST https://<your-n8n>/webhook/randevu \
  -H "Content-Type: application/json" \
  -d '{"sessionId": "visitor-123", "message": "Yarın 14:00 için randevu almak istiyorum"}'
# → {"reply": "Tabii, adınızı ve telefon numaranızı alabilir miyim?"}
```
Each visitor must send a unique `sessionId`; conversation memory is kept per session.

## Setup
1. In n8n: **Workflows → Import from file** → a file from `workflows/`
2. Add credentials and select them on the nodes:
   - Google Calendar OAuth2 (5 calendar tool nodes)
   - OpenAI API (chat model node) — any n8n-supported chat model works
   - Slack (optional, notification node)
3. The calendar is set to `primary`; change it if you use a dedicated calendar
4. Edit the agent's system message for your business hours and holiday dates
5. Activate and open the form URL from the **On form submission** node

## Tech
n8n · LangChain AI Agent · OpenAI · Google Calendar API · Slack · Form Trigger

## Known limitations
- Holiday dates are written for 2026 — religious holidays shift every year and must be updated
- Business rules live in the prompt; a production version should validate them in code as well
- Memory is in-process (window buffer), not persistent

## Credits
Built on top of n8n's community appointment-scheduler pattern, extended with a web form front-end, update/cancel flows, Turkish holiday logic, identity checks and Slack notifications.
