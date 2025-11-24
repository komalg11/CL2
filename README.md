# 🌊 TARANG DSS Engine (Decision Support System)

> **Microservice for Scheme Eligibility, Satellite Asset Integration, and AI Policy Chatbot.**

This module acts as the "Intelligence Layer" of TARANG. It processes data from OCR and Satellite mapping to determine beneficiary eligibility for government schemes (PM-KISAN, PMAY, etc.) and provides an AI-powered chatbot for user support.

---

## 📂 Project Structure

```text
ML_SERVICES/DSS/
├── config/
│   └── database.py       # MongoDB Connection Logic
├── models/
│   └── dss_models.py     # Database Schemas (Village, Claimant, PolicyDoc)
├── routes/
│   └── dss_routes.py     # API Endpoints
├── services/
│   └── dss_services.py   # Business Logic, Rule Engine, RAG AI
├── main.py               # Server Entry Point
├── seed_db.py            # Database Seeder (Runs once to load data)
├── knowledge_base.txt    # Source text for AI Chatbot
├── requirements.txt      # Python Dependencies
└── README.md             # This file
```
## 🛠️ Setup & Installation
### 1. Prerequisites
Python 3.9 or higher
MongoDB Atlas Connection String

### 2. Environment Setup
#### Navigate to DSS folder
cd ML_SERVICES/DSS
#### Create virtual environment
```bash
python -m venv venv

# Windows
.\venv\Scripts\activate

# Mac/Linux
source venv/bin/activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure Secrets
Create a .env file in the root directory (do not commit this file!):
```
MONGO_URL=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/tarang_db?retryWrites=true&w=majority
```

### 5. Initialize the Database
Loads:
Knowledge Base
Village Assets
Clears old test data
```bash
python seed_db.py
```
Expected:
```
✅ DATABASE RESET & SEEDED SUCCESSFULLY!
```

### 6. Run Server
```bash
python main.py
```
API Base URL: http://localhost:8000
Swagger Documentation: http://localhost:8000/docs

# Integration Guide
## Backend (Integration Layer)
### 1. Check Beneficiary Eligibility
Use Case: When a user clicks "Check Status" or submits a form.
Endpoint: POST /api/v1/analyze

Payload:
```json
{
  "claimant_data": { "name": "Raju", "caste": "ST", "age": 45 },
  "asset_data": { "land_size_acres": 2.5, "land_type": "Agricultural", "has_water_source": false },
  "village_id": "Rampur"
}
```
Response: Returns a list of ```recommended_schemes``` and a ```vulnerability_score```

### 2. OCR → DSS Processing
Use Case: When OCR Service finishes scanning a document.
Endpoint: POST /api/v1/analyze/ocr
Payload: Send the raw JSON from the OCR engine. The DSS has an internal adapter to clean it.

## Frontend 
AI Chatbot Integration
Use Case: The floating chat widget.
Endpoint: POST /api/v1/chat

Payload:
```json
{
  "query": "How do I apply?",
  "history": []
}
```
Response Example:
```json
{
  "answer": "To apply, click the 'New Claim' button...",
  "source": "User Manual",
  "confidence": 0.95
}
```
CORS is fully enabled for frontend.

## Satellite / Asset Mapping 
Bulk Asset Update
Use Case: After your U-Net model processes a state/district.
Endpoint: POST /api/v1/assets/batch-update

Payload: List of village objects.
```json
[
  {
    "village_name": "Rampur",
    "water_scarcity_index": 0.9,
    "forest_cover_index": 0.65
  }
]
```
Note: This is a Partial Update. It only updates the fields you send; it won't delete existing Census data.

# Troubleshooting
### 1. "500 Internal Server Error" on /chat?
Cause: The AI memory is empty.
Fix: Run 
```bash
python seed_db.py
```

### 2. "Village Not Found"?
Cause: The village name sent doesn't match any seeded village.
Fix: The system auto-creates a default village entry to prevent crashes, but check spelling (e.g., "Rampur").

### 3. "ModuleNotFoundError"?
Cause: Python can't find the folders.
Fix: Ensure you are running ```python main.py``` from inside the DSS folder, not the root TARANG folder.


