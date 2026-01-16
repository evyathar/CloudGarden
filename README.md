# CloudGarden – IoT + AI Plant Monitoring (Notebook + Gradio)

This repository contains a **Jupyter/Google Colab notebook** (`CloudGarden.ipynb`) that launches a **Gradio** app with multiple tabs for:

- IoT sensor monitoring (dashboard + analytics)
- Daily report generation (DOCX) using an LLM
- Plant disease detection from images (Transformers pipeline)
- Literature Q&A with **RAG** (Retrieval‑Augmented Generation)
- A free multi‑turn chatbot (Gemini)
- A simple gamification/rewards module (stored in Firebase)

---

## What you need to run it successfully

Before you run the notebook, make sure you have:

1) A **Firebase Realtime Database** project
2) A **Gemini API key** (used by **both** Free Chat and RAG generation)
3) (Optional) A **Cerebras API key** (only required for the “Generate Report” tab)
4) A way to provide secrets safely (Colab Secrets or environment variables)

---

## Tabs / Features

- **Realtime Dashboard** – Pulls IoT history from a server and shows plant status + charts.
- **Analytics Dashboard** – KPIs, correlations, distributions, hour/day trends, moving averages.
- **Generate Report** – Produces a daily **English** report and saves it as a **DOCX** file.
- **Plant Disease Detection** – Predicts disease from an uploaded image using a Transformers model.
- **RAG Chat** – Answers from indexed articles (retrieval + LLM generation).
- **Gemini Free Chat** – Multi‑turn chat (session history) using Gemini.
- **Sync Data** – Syncs IoT records from the server into Firebase under `/sensor_data`.
- **Farm Rewards** – Points/tasks/rewards stored in Firebase.

---

## Key concepts (quick definitions)

- **Environment variable**: a key/value configured outside your code (e.g., `GEMINI_API_KEY`) so you don’t hardcode secrets.
- **Colab Secrets**: a Colab feature to store secrets securely for notebooks.
- **Service account JSON**: a Firebase Admin credential file that allows server‑side access via `firebase-admin`.
- **Realtime Database Rules**: Firebase access rules controlling who can read/write paths in the database.
- **RAG (Retrieval‑Augmented Generation)**: first retrieve relevant documents/snippets, then ask the LLM to answer using them.

---

## Secrets / Environment variables

### Gemini (required for Free Chat + RAG)
The notebook tries Colab Secrets first, then environment variables.

**Recommended**
- `GEMINI_API_KEY`

**Fallbacks (the notebook may also check these)**
- `GOOGLE_API_KEY`
- `GENAI_API_KEY` (used as a fallback in some RAG cells)
- `API_KEY` (used as a fallback in some RAG cells)

**Optional**
- `GEMINI_MODEL_ID` – overrides the model name (default is `gemini-2.5-flash`).

### Cerebras (optional – only for Generate Report)
- `CEREBRAS_API_KEY`

> Note: The notebook currently contains an example where `CEREBRAS_API_KEY` is assigned in code. For GitHub/public sharing, remove that and use Secrets/env vars instead.

---

## Firebase setup (Realtime Database)

You need Firebase because the notebook stores and reads:

- IoT data under `/sensor_data`
- RAG indexes under `/indexes/*`
- Gamification state under `/gamification/*`

### Two Firebase access methods used in the notebook

The notebook uses **both** approaches below (important for troubleshooting):

1) **Firebase Admin SDK (`firebase-admin`)**
   - Uses a **service account JSON** file (e.g., `firebase_key.json`).
   - Best for production (authenticated + controlled).

2) **Direct HTTP GET/PUT (`requests`)**
   - Uses `FIREBASE_URL` to read/write JSON directly.
   - Simplest for demos, but will fail if your Realtime Database Rules require authentication.

**Trade‑off summary**
- Admin SDK: more secure, needs the service account file.
- HTTP GET/PUT: easier setup, but can be insecure unless Rules are carefully restricted.

### What to configure

1) **Enable Realtime Database** in your Firebase project.
2) Find your database URL, e.g.:
   - `https://<project-id>-default-rtdb.<region>.firebasedatabase.app/`
3) In the notebook, set:
   - `FIREBASE_URL = "..."`
4) Create a **service account key** and save it as:
   - `firebase_key.json` (same folder as the notebook), or use the notebook’s upload/download logic.

### Realtime Database paths used

- `/sensor_data`
- `/indexes/public_index`
- `/indexes/doc_map`
- `/indexes/doc_text`
- `/gamification/global`

---

## Realtime Database Rules (important for RAG)

Because the RAG indexing code uses **HTTP GET/PUT**, your Rules must allow access to `/indexes/*`.

### Common options (with trade‑offs)

1) **Open read/write globally (demo only)**
   - Easiest, least secure.

2) **Open only `/indexes/*` (better for demos)**
   - RAG works without authentication.
   - Still exposes the indexes publicly.

3) **Require auth everywhere (recommended for real deployments)**
   - Most secure.
   - Requires changing the RAG code to write via Admin SDK or adding auth tokens to HTTP requests.

Example Rules for option (2) (open only `/indexes`):

```json
{
  "rules": {
    "indexes": {
      ".read": true,
      ".write": true
    },
    "sensor_data": {
      ".read": "auth != null",
      ".write": "auth != null"
    },
    "gamification": {
      ".read": "auth != null",
      ".write": "auth != null"
    }
  }
}
```

---

## Running in Google Colab (recommended)

**Why Colab?** It’s the easiest setup (packages, GPU availability, Secrets, and Gradio sharing).

1) Open `CloudGarden.ipynb` in Colab.
2) Run the installation cells (`pip install ...`).
3) Add Colab Secrets:
   - `GEMINI_API_KEY`
   - `CEREBRAS_API_KEY` (only if you use Generate Report)
4) Provide Firebase credentials:
   - Upload `firebase_key.json` when prompted, or use the notebook’s Drive/gdown logic.
5) Confirm `FIREBASE_URL` matches your project.
6) Run the cells in order.
7) Launch the app (usually near the end):

```python
app = build_app()
app.launch(share=True, debug=True)
```

If the Realtime Dashboard is empty, run **Sync Data** first to populate `/sensor_data`.

---

## Running locally

### Option A: Local Jupyter (recommended if you want full control)

1) Create and activate a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\\Scripts\\activate
pip install -U pip
```

2) Install dependencies (exact list may vary by notebook cells):

```bash
pip install -U \
  gradio pandas numpy matplotlib plotly python-docx \
  firebase-admin requests gdown \
  transformers beautifulsoup4 nltk \
  google-genai cerebras-cloud-sdk
```

3) Download NLTK stopwords (first time only):

```bash
python -c "import nltk; nltk.download('stopwords')"
```

4) Set environment variables:

```bash
export GEMINI_API_KEY="YOUR_KEY"
export CEREBRAS_API_KEY="YOUR_KEY"  # only if needed
```

5) Place `firebase_key.json` next to the notebook and set `FIREBASE_URL` in the notebook.

6) Start Jupyter and run the notebook:

```bash
jupyter notebook CloudGarden.ipynb
```

---

## Troubleshooting

- **Error: Missing GEMINI_API_KEY / GOOGLE_API_KEY**
  - Add `GEMINI_API_KEY` in Colab Secrets or export it as an environment variable.

- **Firebase Admin errors (certificate / permissions / databaseURL)**
  - Verify `firebase_key.json` is valid.
  - Ensure Realtime Database is enabled.
  - Make sure the `databaseURL`/`FIREBASE_URL` matches your Firebase project.

- **RAG fails with HTTP 401/403 on GET/PUT**
  - Your Realtime Database Rules likely block unauthenticated access.
  - Use the Rules options above (demo) or update the code to use Admin SDK/auth (production).

- **Disease detection is slow the first time**
  - Transformers models download weights on first run. Ensure internet access.

---

## Security notes (important for GitHub)

- Do **not** commit:
  - `firebase_key.json`
  - any API keys
- Prefer Colab Secrets or `.env` (not committed) for local runs.
- If you currently have keys hardcoded in the notebook, remove them before publishing.

---

## Quick checklist (minimal)

- [ ] `GEMINI_API_KEY` configured (Free Chat + RAG)
- [ ] Firebase Realtime Database enabled
- [ ] `firebase_key.json` available for Admin SDK
- [ ] `FIREBASE_URL` set correctly
- [ ] Realtime Database Rules allow RAG access to `/indexes/*` (or code uses Admin SDK/auth)
- [ ] Launch Gradio with `app.launch(share=True, debug=True)`
