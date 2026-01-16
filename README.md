# CloudGarden (Notebook Gradio App)

CloudGarden is a **Jupyter / Google Colab notebook** that launches a **multi‑tab Gradio web app** for monitoring plant health from IoT sensors and using AI features (chat, RAG, reporting, disease detection).

---

## What this project does

- **IoT monitoring:** visualize temperature / humidity / soil moisture over time and show a simple “plant status”.
- **Analytics:** basic statistics and trend views over the sensor history.
- **RAG (literature Q&A):** ask questions and get answers grounded in retrieved text snippets stored in Firebase.
- **Free chat:** multi‑turn Gemini chat (not tied to the RAG index).
- **Daily report:** generate a DOCX report summarizing a day (LLM-based).
- **Disease detection:** classify a plant image using a pretrained model.
- **Sync + gamification:** sync sensor history into Firebase and store simple rewards/points state.

---

## How it works (high level)

1. The notebook fetches sensor history from an external IoT server endpoint.
2. It stores data in **Firebase Realtime Database** (so the app can load it quickly and persist it).
3. The Gradio UI reads from Firebase and renders charts / summaries.
4. For RAG:
   - literature sources are processed into text
   - an inverted index + mappings are stored in Firebase
   - the app retrieves relevant text and asks Gemini to generate an answer

---

## App tabs (UI)

- **Realtime Dashboard** — recent sensor charts + plant status.
- **Analytics Dashboard** — summary statistics and trends.
- **Generate Report** — DOCX report (LLM).
- **Plant Disease Detection** — image classifier.
- **RAG Chat** — retrieval + Gemini answer.
- **Gemini Chat** — free conversation.
- **Sync Data** — pulls IoT history into Firebase.
- **Farm Rewards** — reads/writes rewards state in Firebase.

---

## Repository structure

- `CloudGarden.ipynb` — main notebook (logic + Gradio UI + launch)
- `README.md` — this file
- `firebase_key.json` — Firebase service account (DO NOT COMMIT)

---

## What you need to run it

### Required
1) **Firebase Realtime Database**
- You must create a Firebase project and enable **Realtime Database**.
- You must provide:
  - the database URL (used by the notebook)
  - a service account key file: `firebase_key.json`

2) **Gemini API key**
- Used by **both**:
  - Free Chat tab
  - RAG Chat generation

### Optional (only for specific tabs)
- **Cerebras API key** — only for the “Generate Report” tab.

---

## Environment variables (set once)

| Variable | Required | Used for |
|---|---:|---|
| `GEMINI_API_KEY` | ✅ | Gemini Chat + RAG Chat |
| `CEREBRAS_API_KEY` | optional | Generate Report tab |

> If you use Colab: store keys in **Colab Secrets** with the same names.

---

## Firebase database paths used

| Path | What it stores |
|---|---|
| `/sensor_data` | synced sensor history |
| `/indexes/public_index` | RAG inverted index (term → doc ids) |
| `/indexes/doc_map` | doc id → source URL |
| `/indexes/doc_text` | extracted text per doc/source |
| `/gamification/global` | rewards/points state |

---

## Quick start (recommended: Colab)

1. Open `CloudGarden.ipynb` in Google Colab.
2. Run the notebook’s **install/dependencies** cell(s).
3. Add `GEMINI_API_KEY` to **Colab Secrets**.
4. Upload your Firebase service account key as `firebase_key.json`.
5. Set your Firebase Realtime Database URL in the notebook configuration (the `FIREBASE_URL` / `databaseURL` constants).
6. Run the launch cell (the notebook calls `build_app()` and `app.launch(...)`).

### Typical first run order inside the UI
1. Open **Sync Data** (to populate `/sensor_data` in Firebase).
2. Go to **Realtime Dashboard** / **Analytics Dashboard**.
3. If you want RAG: build/load the index first, then use **RAG Chat**.

---

## Common issues

- **No data in dashboards:** run **Sync Data** first, and verify the IoT server URL configured in the notebook.
- **RAG index read/write fails (401/403):** your Firebase Realtime DB Rules may block REST access to `/indexes/*`.
- **Chat doesn’t work:** `GEMINI_API_KEY` is missing/incorrect.
- **Report tab errors:** `CEREBRAS_API_KEY` is missing (only needed for that tab).

---

## Security note (important before publishing)

Do not commit:
- `firebase_key.json`
- any API keys

Add them to `.gitignore` and use Secrets/env vars.
