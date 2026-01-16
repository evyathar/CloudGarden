# CloudGarden — IoT + AI Plant Monitoring (Gradio App in a Notebook)

CloudGarden is a **Jupyter/Google Colab notebook** that launches a multi-tab **Gradio** web app for:
- **IoT sensor monitoring** (realtime status + time-series charts)
- **Analytics dashboard** (KPIs, correlations, distributions, hourly/daily patterns, moving averages)
- **Daily DOCX report generation** (English summary via an LLM)
- **Plant disease detection from an image** (HuggingFace `transformers` pipeline)
- **Literature RAG (Retrieval‑Augmented Generation)** backed by a Firebase-hosted index
- **Free multi-turn chat** powered by Gemini
- **Gamification / rewards** stored in Firebase

> **Note:** This project is notebook-first (Colab-friendly). You can also run it locally if you have Jupyter set up.

---

## App tabs (what you will see in the UI)

The notebook defines these tabs in `TABS`:

1. **🌱 Realtime Dashboard**
   Pulls recent IoT history from the server and shows plant status + charts (temperature / humidity / soil / combined).

2. **📊 Analistic Dashboard** *(Analytics)*
   Sensor analytics: statistics cards, correlations, distributions (histograms), hourly/daily patterns, moving averages, etc.

3. **📄 Generate Report**
   Creates a **Word (DOCX)** report summarizing daily sensor data using an LLM (English output).

4. **🖼️ Plant Disease Detection**
   Predicts plant disease/class from an uploaded image using a pretrained `transformers` model.

5. **💬 RAG Chat**
   Answers questions using **retrieved literature snippets** (RAG). The inverted index and document maps are stored in Firebase.

6. **💬 Gemini Chat**
   Free multi-turn chat with per-session history (Gemini).

7. **🔄 Sync Data**
   Syncs IoT history from the server into Firebase under `/sensor_data`.

8. **🎮 Farm Rewards**
   Points/missions/spins/coupons stored in Firebase (auto-refresh on load and on tab select).

---

## High-level architecture

**Data flow (simplified):**

```
IoT Server (Render)  --->  Notebook  --->  Firebase Realtime DB
     /history                |              /sensor_data
                             |
                             +--> RAG Index Builder ---> Firebase /indexes/*
                             |
                             +--> Gradio UI (Tabs) ---> Browser
                             |
                             +--> LLM Providers:
                                   - Gemini (Free Chat + RAG responses)
                                   - Cerebras (DOCX report generation)
```

---

## Repository / project structure

Typical minimal structure:

- `CloudGarden.ipynb` — the main notebook (installs, logic, Gradio UI, launch)
- `README.md` — this file
- `firebase_key.json` — **NOT committed** (Firebase service account; local/Colab only)

---

## Requirements (accounts, services, prerequisites)

### 1) Runtime
Choose one:

- **Option A — Google Colab (recommended)**
  Best experience for sharing Gradio and avoiding local setup issues.

- **Option B — Local (Jupyter)**
  Recommended Python: **3.10+** and stable internet access (models/packages are downloaded).

### 2) Firebase Realtime Database (required)
You need a Firebase project with **Realtime Database** enabled.

The notebook uses Firebase in **two different ways**:

- **A. Firebase Admin SDK** (`firebase-admin`)
  Uses a **service account JSON** (`firebase_key.json`) and writes/reads paths like:
  - `/sensor_data`
  - `/gamification/global`

- **B. Direct REST (HTTP GET/PUT)** (`requests`)
  Used for saving/reading the RAG index under:
  - `/indexes/public_index`
  - `/indexes/doc_map`
  - `/indexes/doc_text`

**Why this matters:**
If your Realtime Database Security Rules block unauthenticated REST access, the RAG index operations will fail (401/403).

### 3) LLM API keys
- **Gemini API key (required for chat features)**
  Used by **both**:
  - **Gemini Chat**
  - **RAG Chat generation** (the “G” in RAG)

- **Cerebras API key (optional)**
  Only required for the **Generate Report** tab.

---

## Environment variables (what you MUST set)

The notebook tries Colab Secrets first (recommended), then falls back to standard environment variables.

| Variable | Required | Used for | Notes |
|---|---:|---|---|
| `GEMINI_API_KEY` | ✅ (for chats) | Gemini Chat + RAG Chat | **Recommended** name (Colab Secret + local env). |
| `GOOGLE_API_KEY` | ⚠️ fallback | Gemini Chat + RAG Chat | Used if `GEMINI_API_KEY` is missing. |
| `GENAI_API_KEY` / `API_KEY` | ⚠️ fallback | RAG Chat | Present as additional fallbacks in the RAG code. |
| `GEMINI_MODEL_ID` | optional | Gemini model override | Default in the notebook: `gemini-2.5-flash`. |
| `CEREBRAS_API_KEY` | ✅ only if using reports | Generate Report | Do **not** hard-code in the notebook before sharing. |

### Colab: how to set keys
Use **Colab Secrets** (recommended), and create:
- `GEMINI_API_KEY`
- `CEREBRAS_API_KEY` *(only if you use the report tab)*

### Local: how to set keys
Linux/macOS:
```bash
export GEMINI_API_KEY="YOUR_GEMINI_KEY"
export CEREBRAS_API_KEY="YOUR_CEREBRAS_KEY"  # optional
```

Windows (PowerShell):
```powershell
setx GEMINI_API_KEY "YOUR_GEMINI_KEY"
setx CEREBRAS_API_KEY "YOUR_CEREBRAS_KEY"  # optional
```

---

## Configuration you may need to change (constants inside the notebook)

These are not environment variables by default (they are defined in code cells).
If you fork the project, you typically update:

- `FIREBASE_URL` — your Realtime Database URL (used by REST GET/PUT for RAG)
- `firebase_key.json` — service account file for Admin SDK
- `BASE_URL` — IoT server base URL (Render), used for `/history`
  - Default found in the notebook: `https://server-cloud-v645.onrender.com/`
- `FEED` — server feed name (default: `"json"`)
- `BATCH_LIMIT` — default: `200`
- `REPORT_MODEL_NAME` — report LLM name (default in notebook: `llama3.1-8b`)
- `DOC_URLS` — list of sources/DOIs/URLs used to build the RAG index
- `MODEL_NAME` (disease detection) — HuggingFace model id used by the pipeline

---

## Quickstart: run in Google Colab (recommended)

1. Open `CloudGarden.ipynb` in Colab.
2. Run the **install** cell (`pip install ...`).
3. Add Colab Secrets:
   - `GEMINI_API_KEY`
   - `CEREBRAS_API_KEY` *(optional)*
4. Provide Firebase Admin credentials:
   - Upload a **service account JSON** as `firebase_key.json`
     *(or use your Drive + `gdown` flow if your notebook supports it).* 
5. Verify Firebase URLs:
   - `FIREBASE_URL` (REST)
   - `databaseURL` in `firebase_admin.initialize_app(...)`
6. Run the notebook cells in order.
7. Launch the app (the notebook has a single launch cell), e.g.:

```python
app = build_app()
app.launch(share=True, debug=True)
```

---

## Quickstart: run locally (Jupyter)

### 1) Create a virtual environment
```bash
python -m venv .venv
# macOS/Linux:
source .venv/bin/activate
# Windows:
# .venv\\Scripts\\activate
pip install -U pip
```

### 2) Install dependencies
Your notebook may already contain an install cell; for local stability, the common packages used include:

```bash
pip install -U \
  gradio pandas numpy matplotlib plotly python-docx \
  firebase-admin requests beautifulsoup4 nltk \
  transformers google-genai \
  cerebras-cloud-sdk gdown
```

If NLTK stopwords are used:
```bash
python -c "import nltk; nltk.download('stopwords')"
```

### 3) Add secrets and Firebase key
- Set env vars (see the Environment variables section).
- Place `firebase_key.json` next to the notebook (do not commit it).

### 4) Run the notebook
```bash
jupyter notebook CloudGarden.ipynb
```
Then execute cells until the launch cell.

---

## Firebase paths used by the project

| Path | Purpose |
|---|---|
| `/sensor_data` | Synced sensor history from the IoT server |
| `/indexes/public_index` | Inverted index for RAG (term → list of doc_ids) |
| `/indexes/doc_map` | doc_id → source URL |
| `/indexes/doc_text` | extracted text per document/source |
| `/gamification/global` | points/missions/spins/coupons state |

> You may see additional `/indexes/*` keys from earlier iterations (e.g., `doc_meta`, `docTexts`). The main pipeline uses the three listed above.

---

## Security notes (important before sharing on GitHub)

- **Never commit**:
  - `firebase_key.json`
  - API keys (Gemini/Cerebras)
- The notebook currently supports **unauthenticated REST PUT/GET** for `/indexes/*`.
  This works only if your Firebase Rules allow it.

### Firebase Rules options (trade-offs)
1. **Fastest demo (not recommended):** open read/write globally
   - ✅ works instantly
   - ❌ insecure

2. **Better demo:** open only `/indexes/*`
   - ✅ RAG works
   - ✅ limits exposure
   - ❌ still public index

3. **Recommended for production:** avoid unauthenticated REST writes
   - ✅ secure
   - ✅ consistent auth
   - ❌ requires code changes (use Admin SDK or authenticated REST)

Example Rules for option (2):
```json
{
  "rules": {
    "indexes": {
      ".read": true,
      ".write": true
    }
  }
}
```

---

## Troubleshooting

- **“Missing GEMINI_API_KEY …”**
  Add `GEMINI_API_KEY` to Colab Secrets or set it as an environment variable.

- **Firebase initialization errors (credentials / databaseURL)**
  Ensure `firebase_key.json` is valid and your `databaseURL` points to your Realtime Database.

- **RAG fails with 401/403 during REST GET/PUT**
  This is almost always Firebase Rules. See the Security notes section.

- **Disease detection is slow the first time**
  The `transformers` model downloads weights on first run. Ensure stable internet.

- **IoT dashboards show “no data”**
  The IoT server may be unavailable, the `BASE_URL` may be wrong, or you didn’t run **Sync Data** yet.

---

## License / usage

This is an academic notebook-style project. If you publish it, remove any hard-coded secrets and keep credentials out of the repository.
