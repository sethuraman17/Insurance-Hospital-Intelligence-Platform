🏥 Insurance–Hospital Intelligence Platform

(Supabase + LangChain + LangGraph + FAISS)

This project is a real-world AI system that answers insurance and hospital questions such as:

“Is KMCH Coimbatore cashless for Bajaj Allianz and is cataract covered?”

by combining:

Verified hospital–insurer data (Supabase SQL)

Policy clauses from PDFs (FAISS + SentenceTransformers)

LLM reasoning (Groq / LLaMA)

LangGraph for orchestration and memory

It is designed to avoid hallucinations, respect verification status, and clearly separate facts from policy interpretation.

🧠 System Architecture
┌─────────────────────────┐
│        User Query       │
└────────────┬────────────┘
             │
     ┌───────▼────────┐
     │ Intent Router  │
     │  (LLM / rules) │
     └───────┬────────┘
             │
   ┌─────────▼──────────┐        ┌───────────────┐
   │ SQL Retriever      │        │ PDF Retriever │
   │ (Supabase Views)   │        │ (FAISS + ST)   │
   └─────────┬──────────┘        └───────┬───────┘
             │                           │
      ┌──────▼──────────────┐    ┌──────▼──────────────┐
      │  Fact Normalizer     │    │  Clause Selector    │
      └─────────┬───────────┘    └─────────┬───────────┘
                │                            │
           ┌────▼────────────────────────────▼────┐
           │        Answer Composer (LLM)           │
           │  (facts + policy text + confidence)   │
           └───────────────────────────────────────┘


LangGraph executes this pipeline as a stateful decision graph with memory.

📦 Data Sources
1️⃣ Structured Data (Supabase)

A fully normalized relational model stores:

Hospitals

Districts

Insurance companies

Policies

Hospital–insurer empanelments

Verification status

Policy mapping

Audit trails

All chatbot queries use read-only database views, such as:

hospital_empanelment_view

policy_view

verification_view

This guarantees:

No numeric IDs

Human-readable data

No accidental writes

Strong referential integrity

2️⃣ Policy Documents (PDF → FAISS)

Official insurance PDFs are parsed and converted into a custom FAISS vector index:

Insurer	Document
HDFC ERGO	Total Health Plan Policy
HDFC ERGO	Optima Secure Policy
ICICI Lombard	Health Insurance Brochure
Bajaj Allianz	Silver Health Policy

These are:

Chunked

Embedded using SentenceTransformers

Stored in FAISS

Grouped by policy type

The system never uses LangChain’s vectorstore.as_retriever() — it uses direct FAISS + embeddings for full control.

🧩 LangGraph Design

The chatbot is implemented as a LangGraph state machine.

State includes:

user_query

intent

sql_facts

retrieved_clauses

final_answer

chat_history

Nodes:
Node	Responsibility
intent	Detect structured vs document vs hybrid query
sql_retriever	Query Supabase views
pdf_retriever	Retrieve policy clauses from FAISS
join	Merge parallel results
adjudicator	LLM that reasons over both sources
memory	Stores conversation history

The graph supports parallel SQL + PDF execution for hybrid questions.

🔍 How a Query Is Answered

Example:

“Is KMCH cashless for Bajaj Allianz and is cataract covered?”

Step 1 – Intent Routing

Detected as hybrid (hospital + coverage).

Step 2 – SQL Retrieval (Supabase)

Finds:

KMCH – Bajaj Allianz
cashless = true
verified = true

Step 3 – PDF Retrieval (FAISS)

Finds policy clauses about:

Day care procedures

Cataract surgery

Waiting periods

Exclusions

Step 4 – Adjudication

The LLM:

Uses SQL as the source of truth

Uses PDFs for legal interpretation

Adds verification & confidence

Avoids hallucinations

Final Answer:

A combined, fact-based, policy-aware explanation.

🛡 Why This System Is Safe
Risk	How it is prevented
Hallucinated hospitals	Only SQL views allowed
Fake insurer relationships	FK + verified flags
Wrong coverage claims	Must be supported by PDF text
Overconfidence	Verified vs pending status surfaced
LLM lying	SQL is authoritative
🧪 Technologies Used

Supabase (PostgreSQL + Views + RLS)

LangChain (LLM, prompt orchestration)

LangGraph (stateful agent workflow)

SentenceTransformers

FAISS

Groq (LLaMA-3.1)

Python

🚀 Why This Is Not a Toy Project

This system implements:

Real insurer & hospital data

Real policy documents

Legal-style reasoning

Verification & audit trails

Multi-source fact synthesis

It is architected like a real insurance backend, not a demo chatbot.

📌 Future Extensions

Multi-hospital comparisons

Claim eligibility scoring

Pre-authorization simulation

User-uploaded policy PDFs

Regulatory compliance checks

🏁 Summary

This project is a production-grade AI decision system for health insurance and hospital networks, built on:

SQL for truth + PDFs for law + LangGraph for reasoning

It demonstrates how LLMs can be safely used in regulated domains like healthcare and insurance.
