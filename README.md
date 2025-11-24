# 🧠 AI RAG Engine (Retrieval-Augmented Generation)

> **Note:** The RAG Logic has been integrated directly into the **[DSS Microservice](../DSS)** to ensure zero-latency responses and simplified deployment.

This module defines the architecture for the **AI Policy Chatbot** used in TARANG. It provides instant, accurate answers regarding FRA Laws, Government Schemes, and Technical Support.

---

## 🏗️ Architecture

We utilize a **Lightweight, Server-Side RAG Pipeline** optimized for speed and accuracy without heavy cloud dependencies.

### 1. Embedding Model
* **Model:** `all-MiniLM-L6-v2` (Sentence-Transformers)
* **Function:** Converts user queries and policy documents into 384-dimensional dense vectors.
* **Why:** High speed, low memory usage, and excellent semantic understanding of Indian government terminology.

### 2. Knowledge Base
* **Source:** Curated `knowledge_base.txt` containing:
    * **FRA 2006 Laws** (Claims, Rights, Evidence)
    * **Central Schemes** (PM-KISAN, PMAY, MGNREGA)
    * **State Schemes** (Odisha, MP, Telangana, Tripura)
    * **Technical Support** (Portal navigation, Asset Mapping)
* **Ingestion:** The `seed_db.py` script chunks this text and generates vectors.

### 3. Retrieval Engine
* **Method:** Cosine Similarity (via Dot Product calculation).
* **Storage:** Vectors are stored in **MongoDB Atlas** (`PolicyDoc` collection).
* **Caching:** On startup, the DSS loads vectors into **In-Memory Cache** for O(1) retrieval speed.

### 4. Smart Fallback
* **Confidence Threshold:** If the similarity score is `< 0.25`, the system identifies the query as "Out of Domain."
* **Action:** Instead of hallucinating, it returns verified **MoTA Helpline Numbers** and **Email IDs**.

---

## 🔄 Workflow

1.  **User Query:** "How do I get a Patta?"
2.  **Vectorization:** Converted to `[0.01, -0.23, ...]`
3.  **Search:** Compared against 50+ cached policy vectors.
4.  **Match:** Finds "FRA Claim Process" section (Score: 0.89).
5.  **Response:** Returns the specific text chunk + Source Citation.

---

## 🚀 Future Roadmap (Scalability)

While the current In-Memory approach is perfect for the prototype (<10k docs), the production roadmap includes:
* **Vector Database:** Migration to **pgvector** or **Qdrant** for million-scale document search.
* **LLM Integration:** Connecting a Generative Model (e.g., Llama-2, GPT-4) to synthesize natural language answers from the retrieved chunks.
* **Multi-Lingual Support:** Fine-tuning embeddings for Hindi, Odia, and Telugu.
