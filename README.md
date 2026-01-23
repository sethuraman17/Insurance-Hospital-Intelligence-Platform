# 🏥 Insurance–Hospital Intelligence Platform  
**Supabase • LangChain • LangGraph • FAISS • LLM**

This project is a production-grade AI system that answers complex insurance and hospital queries such as:

> “Is KMCH Coimbatore cashless for Bajaj Allianz and is cataract covered?”

by combining:
- **Verified hospital–insurer data (Supabase SQL)**
- **Insurance policy clauses from PDFs (FAISS + embeddings)**
- **LLM reasoning (Groq / LLaMA)**
- **LangGraph orchestration with memory**

<img width="1247" height="663" alt="image" src="https://github.com/user-attachments/assets/0a63e382-3047-4445-bfb7-6c024f0411f4" />

The system is designed to **avoid hallucinations**, respect **verification status**, and clearly separate **facts from policy interpretation**.

---


LangGraph executes this flow as a **stateful graph** with memory and branching.

---

## 📦 Data Sources

### 1. Structured Data (Supabase)
A normalized relational schema stores:
- Hospitals
- Districts
- Insurance companies
- Policies
- Hospital–insurer empanelments
- Cashless status
- Verification status
- Audit logs

All chatbot queries use **read-only views**:
- `hospital_empanelment_view`
- `policy_view`
- `verification_view`

This ensures:
- Human-readable output
- No accidental writes
- Strong referential integrity

---

### 2. Policy Documents (PDF → FAISS)
The system uses official insurer PDFs:

| Insurer | Document |
|-------|---------|
| HDFC ERGO | Total Health Plan |
| HDFC ERGO | Optima Secure |
| ICICI Lombard | Health Insurance Brochure |
| Bajaj Allianz | Silver Health Policy |

These PDFs are:
- Parsed
- Chunked
- Embedded with `SentenceTransformers`
- Indexed in **FAISS**
- Grouped by policy type

The system uses **custom FAISS retrieval**, not LangChain’s default retrievers.

---

## 🧩 LangGraph Design

The chatbot runs as a **LangGraph state machine**.

### State includes:
- `user_query`
- `intent`
- `sql_facts`
- `retrieved_clauses`
- `answer`
- `chat_history`

### Graph Nodes:
| Node | Function |
|------|--------|
| `intent` | Detect structured vs document vs hybrid query |
| `sql_retriever` | Query Supabase views |
| `pdf_retriever` | Retrieve policy clauses from FAISS |
| `join` | Merge parallel SQL + PDF results |
| `adjudicator` | LLM that reasons over both |
| `memory` | Stores conversation history |

Hybrid questions run SQL and PDF retrieval **in parallel**.

---

## 🔍 How a Query Is Answered

Example:
> “Is KMCH cashless for Bajaj Allianz and is cataract covered?”

1. **Intent detection** → Hybrid  
2. **SQL lookup** → KMCH × Bajaj Allianz, cashless = true, verified = true  
3. **PDF retrieval** → Cataract coverage clauses  
4. **LLM adjudication** → Combines facts + clauses  
5. **Final answer** → With verification & confidence

---

## 🛡 Safety & Reliability

| Risk | How it is prevented |
|------|-------------------|
Hallucinated hospitals | Only Supabase views are used |
Fake insurer relationships | Foreign keys + verification flags |
Wrong coverage claims | Must be supported by PDF text |
Overconfidence | Verified vs pending is surfaced |
LLM guessing | SQL is treated as the source of truth |

<img width="1333" height="636" alt="image" src="https://github.com/user-attachments/assets/caf63b17-5f8e-4a5e-8bc7-178c8e47e1e5" />

---

## 🧪 Technology Stack

- **Supabase (PostgreSQL, Views, RLS)**
- **LangChain**
- **LangGraph**
- **SentenceTransformers**
- **FAISS**
- **Groq (LLaMA-3.1)**
- **Python**

---

## 🚀 Why This Is Not a Toy Project

This system uses:
- Real hospital and insurer data
- Real policy documents
- Legal-style policy reasoning
- Verification & audit trails
- Multi-source fact synthesis

It is architected like a **real insurance backend**, not a demo chatbot.

---

## 📌 Future Extensions

- Claim eligibility simulation
- Pre-authorization checks
- Multi-hospital comparisons
- Upload-your-policy analysis
- Regulatory compliance checking

---

<img width="609" height="363" alt="image" src="https://github.com/user-attachments/assets/c2152c1e-cda0-4c53-a3ac-65a5a485f563" />

## 🏁 Summary

This project demonstrates how to safely combine:

> **SQL for truth + PDFs for law + LangGraph for reasoning**

to build an AI system suitable for **regulated domains** like healthcare and insurance.
