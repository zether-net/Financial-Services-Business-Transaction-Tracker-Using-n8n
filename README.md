# Financial-Services-Business-Transaction-Tracker-Using-n8n
Automated ingestion, OCR, parsing, profit-calculation, Google Sheets bookkeeping, and Telegram notification for your GCash / SeaBank transaction screenshots.


## ✨ What this workflow does

| Stage | Purpose | Key node(s) |
|-------|---------|-------------|
| **1. Receive** | Accept a `POST` request at `/gcash-transactions` carrying a screenshot (Base-64) and raw OCR text. | **Webhook** (`gcash-transactions`) |
| **2. Extract image** | Convert the Base-64 string into a file so Vision OCR can read it. | **Convert to File** |
| **3. OCR** | Run GPT-4o Vision to pull every legible line of text from the screenshot. | **Analyze Image** |
| **4. Classify** | Detect the platform (GCash / SeaBank) and transaction type (Cash In, Cash Out, Telco Load …). | **IF** + **Set** nodes |
| **5. AI agent** | Parse amount, date, reference #, account #, normalise numbers, apply the 2 % / ₱5-minimum profit rule, and decide **append vs update**. | **AI Agent** (LangChain → OpenAI Chat) |
| **6. Database** | Upsert to Google Sheets (`Transactions` tab) keyed by `reference_number`. | **Google Sheets: Append / Update Row** |
| **7. Notify / Edit** | Send a Telegram message summarising the entry and open an inline form for corrections. | **Telegram: Send Message & Edit Form** |
| **8. Respond** | Return a pipe-delimited response to the caller (great for debugging Shortcuts). | **Respond to Webhook** |

---

## 📐 Architecture at a glance
```mermaid
graph TD
  A[Client / Shortcut] -->|POST JSON| B(🔗 Webhook<br>/gcash-transactions)
  B --> C(🗂 Convert to File)
  C --> D(🧠 Vision OCR)
  D --> E{🔎 IF<br>GCash? SeaBank?<br>Type?}
  E --> F(📦 Set fields)
  F --> G(🤖 AI Agent<br>Parse + Upsert decision)
  G --> H(📊 Google Sheets<br>Append / Update)
  H --> I(📨 Telegram Bot<br>Notify + Inline Edit)
  I --> J(🔄 Webhook Response)

