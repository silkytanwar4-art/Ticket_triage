# 🎫 Ticket Triage — Automated AI Resolution Pipeline

> Fetch → Clean → Prioritise → Resolve with AI → Store in MongoDB


---

## 📌 Overview

**Ticket Triage** is an end-to-end n8n automation workflow that:

1. Downloads a real customer support dataset from **Kaggle**
2. Cleans and normalises the ticket data via a **Python microservice**
3. Assigns a numeric **urgency score** based on ticket priority
4. Generates a professional **AI-powered resolution** for each ticket using **Ollama (LLaMA 3.2)**
5. Stores every ticket and its resolution in **MongoDB**

No manual effort. One click runs the entire pipeline.

---

## 🔄 Pipeline Flow

```
Manual Trigger
      │
      ▼
HTTP Request ──────────── Kaggle API (download dataset)
      │
      ▼
Python Microservice ────── Clean binary rows (port 5002)
      │
      ▼
Limit ─────────────────── Cap at 10 tickets per run
      │
      ▼
Code (Python) ─────────── Normalise text · map priority → urgency score
      │
      ▼
Loop Over Items
      │                   ┌─────────────────┐
      ├──────────────────▶│  Ollama LLaMA   │
      │                   │  llama3.2:1b    │
      │                   └────────┬────────┘
      │                            │
      │                   ┌────────▼────────┐
      │                   │   LLM Chain     │
      │                   │ Generate fix    │
      │                   └────────┬────────┘
      │◀───────────────────────────┘
      │
      ▼
MongoDB ────────────────── Insert ticket + AI resolution
```

---

## 🧩 Node Reference

| # | Node | Type | Role |
|---|------|------|------|
| 1 | Manual Trigger | Trigger | Starts the workflow on demand |
| 2 | HTTP Request | HTTP | Downloads the Kaggle customer support dataset |
| 3 | Python Microservice | HTTP (POST) | Sends binary data to `python_scrub:5002` for cleaning |
| 4 | Limit | Control | Caps processing to **10 tickets** per execution |
| 5 | Code (Python) | Code | Cleans descriptions · replaces placeholders · maps priority → urgency |
| 6 | Edit Fields | Set | Structures ticket fields for the loop |
| 7 | Loop Over Items | Split in Batches | Iterates one ticket at a time |
| 8 | Ollama Chat Model | LLM | Runs `llama3.2:1b` locally |
| 9 | Basic LLM Chain | LangChain | Prompts the model and extracts resolution text |
| 10 | Edit Fields 1 | Set | Merges ticket data with the AI resolution |
| 11 | Insert Documents | MongoDB | Saves the complete record to the `customer tickets` collection |

---

## 🤖 AI Resolution

The **LLM Chain** sends a structured prompt to Ollama for each ticket.

**System role:**
> *"You are a professional customer support resolution expert. Provide a clear, helpful, and concise resolution. Respond with only the Resolution text — no greetings, no formatting."*

**Inputs passed to the model:**

- Customer Name & Email
- Product Purchased
- Ticket ID, Type & Description
- Ticket Priority & Urgency Score

---

## 🐍 Data Cleaning Logic

The `Code (Python)` node normalises each ticket before it reaches the AI:

| Operation | Detail |
|-----------|--------|
| Strip newlines | Removes `\n` and `\\n` from descriptions |
| Replace placeholder | `{product_purchased}` → actual product name |
| Sanitise quotes | Converts `"` to `'` to prevent JSON issues |
| Collapse whitespace | Multiple spaces → single space |
| Map priority to urgency | See table below |

**Priority → Urgency mapping:**

| Ticket Priority | Urgency Score |
|----------------|:-------------:|
| Critical | 1 |
| High | 2 |
| Medium | 3 |
| Low | 4 |
| Very Low | 5 |

---

## 🗄️ MongoDB Output Schema

Each document inserted into the `customer tickets` collection:

```json
{
  "Customer Name":      "Jane Smith",
  "Customer Email":     "jane@example.com",
  "Product Purchased":  "Sony Xperia",
  "Ticket ID":          "T-1042",
  "Ticket Description": "My device stopped charging after the update.",
  "Ticket Priority":    "High",
  "Urgency":            2,
  "Resolution":         "Please try a factory reset after backing up your data..."
}
```

---

## ⚙️ Prerequisites

| Requirement | Details |
|-------------|---------|
| **n8n** | Any recent version, self-hosted or cloud |
| **Ollama** | Running locally with `llama3.2:1b` pulled |
| **MongoDB** | Instance accessible from n8n |
| **python_scrub** | Python microservice running on port `5002` |
| **Kaggle API key** | Added to the Authorization header in HTTP Request node |

---

## 🔑 Credentials

| Service | n8n Credential Name | Node |
|---------|-------------------|------|
| Kaggle API | `Authorization` header | HTTP Request |
| Ollama | `Ollama account` | Ollama Chat Model |
| MongoDB | `MongoDB account` | Insert Documents |

---

## 🚀 Quick Start

```bash
# 1. Pull the LLaMA model
ollama pull llama3.2:1b

# 2. Start your Python microservice
python app.py  # must listen on port 5002

# 3. Import the workflow into n8n
#    File → Import → select customer_tickets.json

# 4. Add credentials in n8n Settings → Credentials

# 5. Open the workflow and click "Execute Workflow"
```

---

## 📁 Repository Structure

```
Ticket_triage/
├── .github/
│   └── workflows/
│       └── main.yml          # GitHub Actions: auto-validates all JSON files
├── customer_tickets.json     # n8n workflow definition (import this into n8n)
└── README.md
```

---

## 🛠️ GitHub Actions

Every push to `main` automatically validates all `.json` files in the repo:

```
Triggers on every push or pull request to main
Scans all *.json files recursively
✅ Green tick = valid JSON
❌ Red cross = fix the JSON
```

To reuse this workflow in any other repository — just copy `.github/workflows/main.yml`. No changes needed.

---

## 👤 Author

**silkytanwar4-art**

> Built with n8n · Ollama · MongoDB · Python
