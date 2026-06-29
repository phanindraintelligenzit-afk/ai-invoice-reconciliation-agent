# AI Invoice Reconciliation Agent

> **Automate your Accounts Payable. Match invoices to POs in seconds. Catch discrepancies before they cost you money.**

The AI Invoice Reconciliation Agent is a production-ready, multi-agent pipeline that ingests invoices from PDF, email, and EDI sources, matches them against purchase orders and receipts, flags discrepancies, and routes exceptions to the right approver — all without manual data entry.

---

## 🚀 Why This Exists

### The Problem

Accounts Payable teams are drowning in manual work:

- **Manual AP workflows** — Staff spend hours keying invoice data, cross-referencing POs, and chasing approvers.
- **Invoice-PO matching errors** — Human-led matching leads to overpayments, duplicate payments, and missed early-payment discounts.
- **Discrepancy detection gaps** — Price mismatches, missing line items, and duplicate invoices slip through, causing audit findings and financial leakage.

### The Solution

A **4-agent AI pipeline** that handles the entire invoice-to-reconciliation lifecycle:

| Agent | Responsibility |
|-------|---------------|
| **Document Parser** | Extracts line items, totals, dates, and vendor info from invoices arriving via **PDF, email, or EDI**. |
| **PO Matcher** | Matches each invoice to the correct **purchase order and receiving receipt** using fuzzy and rule-based matching. |
| **Discrepancy Detector** | Flags **price mismatches, missing items, duplicate invoices**, and quantity variances against PO/receipt data. |
| **Approval Router** | Routes exceptions to the **correct approver** based on configurable rules (amount thresholds, vendor, department). |

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                     AI Invoice Reconciliation Agent                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│  │  PDF / Email │───▶│  Document    │───▶│  Structured Invoice  │  │
│  │  / EDI       │    │  Parser      │    │  Data (JSON)         │  │
│  └──────────────┘    └──────────────┘    └──────────┬───────────┘  │
│                                                      │              │
│                                                      ▼              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│  │  Purchase    │───▶│  PO Matcher  │◀───│  Invoice Data        │  │
│  │  Orders &    │    │  (fuzzy +    │    │                      │  │
│  │  Receipts    │    │   rules)     │    └──────────────────────┘  │
│  └──────────────┘    └──────┬───────┘                              │
│                              │                                      │
│                              ▼                                      │
│                     ┌────────────────┐                              │
│                     │  Discrepancy   │                              │
│                     │  Detector      │                              │
│                     │  (price, qty,  │                              │
│                     │   duplicates)  │                              │
│                     └───────┬────────┘                              │
│                              │                                      │
│              ┌───────────────┼───────────────┐                      │
│              ▼                               ▼                      │
│     ┌────────────────┐            ┌────────────────────┐           │
│     │  Auto-Approve  │            │  Approval Router   │           │
│     │  (clean match) │            │  (rules-based      │           │
│     └────────────────┘            │   exception flow)  │           │
│                                   └────────┬───────────┘           │
│                                            │                       │
│                                            ▼                       │
│                                   ┌────────────────────┐           │
│                                   │  Approver /        │           │
│                                   │  ERP / Accounting  │           │
│                                   │  System            │           │
│                                   └────────────────────┘           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🛠 Tech Stack

| Layer | Technology |
|-------|-----------|
| **Language** | Python 3.11+ |
| **AI / LLM** | OpenAI GPT-4 / Anthropic Claude (for document understanding) |
| **Document Parsing** | PyPDF2, pdfplumber, Tesseract OCR, python-docx |
| **Data Processing** | Pandas, NumPy, Pydantic |
| **Matching Engine** | FuzzyWuzzy / RapidFuzz, custom rule engine |
| **API Framework** | FastAPI |
| **Task Queue** | Celery + Redis |
| **Database** | PostgreSQL |
| **Deployment** | Docker, Docker Compose |
| **Monitoring** | Prometheus, Grafana (optional) |

---

## 📦 Installation

### Prerequisites

- Python 3.11+
- Docker & Docker Compose (recommended)
- Redis
- PostgreSQL 14+

### Clone & Setup

```bash
# Clone the repository
git clone https://github.com/phanindraintelligenzit-afk/ai-invoice-reconciliation-agent.git
cd ai-invoice-reconciliation-agent

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env with your API keys and database credentials
```

### Docker (Quick Deploy)

```bash
docker-compose up -d
```

---

## ⚡ Quick Start

```bash
# 1. Start the API server
uvicorn app.main:app --reload --port 8000

# 2. Submit an invoice for processing
curl -X POST http://localhost:8000/api/v1/invoices/submit \
  -H "Content-Type: multipart/form-data" \
  -F "file=@sample_invoice.pdf" \
  -F "source=pdf"

# 3. Check reconciliation status
curl http://localhost:8000/api/v1/invoices/{invoice_id}/status

# 4. View flagged discrepancies
curl http://localhost:8000/api/v1/discrepancies?status=open
```

### Python SDK Example

```python
from ai_invoice_reconciliation import InvoiceClient

client = InvoiceClient(api_key="your-api-key")

# Submit an invoice
result = client.submit_invoice("invoice_001.pdf")

# Get reconciliation result
reconciliation = client.get_reconciliation(result.invoice_id)

print(f"Status: {reconciliation.status}")
print(f"Matched PO: {reconciliation.po_number}")
print(f"Discrepancies: {reconciliation.discrepancies}")
```

---

## 📡 API Endpoints

### Invoices

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/invoices/submit` | Upload and submit an invoice (PDF/EDI/email) |
| `GET` | `/api/v1/invoices` | List all invoices with optional filters |
| `GET` | `/api/v1/invoices/{id}` | Get invoice details and extracted data |
| `GET` | `/api/v1/invoices/{id}/status` | Check processing/reconciliation status |
| `DELETE` | `/api/v1/invoices/{id}` | Remove an invoice from the system |

### Reconciliation

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/reconciliation/run` | Trigger reconciliation for an invoice |
| `GET` | `/api/v1/reconciliation/{id}` | Get reconciliation results |
| `GET` | `/api/v1/reconciliation/summary` | Get reconciliation summary/stats |

### Discrepancies

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/discrepancies` | List all discrepancies (filter by status/type) |
| `GET` | `/api/v1/discrepancies/{id}` | Get discrepancy details |
| `POST` | `/api/v1/discrepancies/{id}/resolve` | Mark a discrepancy as resolved |

### Approvals

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/approvals` | List pending approval items |
| `POST` | `/api/v1/approvals/{id}/approve` | Approve an exception |
| `POST` | `/api/v1/approvals/{id}/reject` | Reject an exception |
| `GET` | `/api/v1/approvals/queue/{approver_id}` | Get approver's queue |

### Purchase Orders

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/purchase-orders` | Create/import a purchase order |
| `GET` | `/api/v1/purchase-orders` | List purchase orders |
| `GET` | `/api/v1/purchase-orders/{id}` | Get PO details |

---

## 💼 Use Cases

### 🏢 Accounts Payable Teams
Eliminate manual data entry and reduce invoice processing time from days to minutes. Let your team focus on exceptions, not routine matching.

### 💰 Finance Departments
Gain real-time visibility into invoice status, outstanding discrepancies, and approval bottlenecks. Improve cash flow forecasting with accurate AP data.

### 📊 Accounting Firms
Offer AI-powered reconciliation as a value-added service to clients. Handle higher volumes without adding headcount, and deliver cleaner audit-ready books.

---

## 💵 Pricing

| Component | Cost |
|-----------|------|
| **This Repository** | **Free & Open Source (MIT)** — clone, fork, customize. |
| **Setup & Deployment** | **$3,000 – $10,000** one-time setup fee depending on complexity (ERP integration, custom rules, on-prem vs. cloud). |
| **Custom Development** | Available on request — custom connectors, additional document types, bespoke matching rules. |

> 📧 Contact us for a custom quote tailored to your AP volume and integration needs.

---

## 🗺 Roadmap

- [x] Document Parser (PDF, email, EDI ingestion)
- [x] PO Matcher (fuzzy + rule-based matching)
- [x] Discrepancy Detector (price, quantity, duplicates)
- [x] Approval Router (rules-based exception routing)
- [ ] ERP connectors (SAP, Oracle, NetSuite, QuickBooks)
- [ ] Web dashboard for AP teams
- [ ] Multi-language invoice support
- [ ] ML-based anomaly detection for fraud prevention
- [ ] Slack/Teams approval notifications
- [ ] Batch processing for high-volume environments
- [ ] Audit trail & compliance reporting

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 🤖 AIdentify

Built with ❤️ by the **AIdentify** team — AI-powered automation for modern finance operations.

[Website](https://aidentify.com) · [GitHub](https://github.com/aidentify) · [Contact](mailto:hello@aidentify.com)
