# VidQuery

> An intelligent meeting intelligence and video understanding platform that transcribes, extracts actionable insights, summarizes, and enables conversational retrieval (RAG) over meeting recordings and YouTube videos using OpenAI Whisper, Sarvam AI, Mistral AI, ChromaDB, and Streamlit.

---

## 1. Title and One-Line Summary

**VidQuery** transforms English and Hinglish video/audio recordings into structured executive summaries, categorized action items, key decisions, and an interactive Q&A assistant using OpenAI Whisper, Sarvam AI, LangChain LCEL, Mistral AI, and ChromaDB.

---

## 2. Problem and Users

### The Problem
Professionals, students, and teams spend hours watching video recordings, webinars, lectures, and meeting replays to locate key discussions, task assignments, and decisions. Manually taking notes or scanning lengthy raw transcripts is slow, error-prone, and context-heavy. Additionally, meetings conducted in Indian corporate environments frequently code-switch between Hindi and English ("Hinglish"), causing standard monolingual speech-to-text models to fail or produce garbled outputs.

### Target Users
- **Product & Project Managers:** Need to quickly pull action items, assigned owners, deadlines, and agreed decisions after stakeholder calls.
- **Engineers & Technical Teams:** Need to verify technical architecture discussions and ask specific questions about decisions made during design reviews.
- **Students & Researchers:** Need to ingest recorded lectures or YouTube tutorials and ask targeted questions without re-watching multi-hour videos.

### What It Replaces or Improves
- **Replaces:** Manual meeting note-taking, manual playback searching, and expensive manual transcription services.
- **Improves:** Standard automated transcription tools by adding native Hinglish translation support, hierarchical map-reduce summarization to bypass LLM context limits, structured action item extraction, and grounded RAG search that avoids hallucinating unmentioned facts.

---

## 3. Timeline

| Metric | Detail | Source |
| :--- | :--- | :--- |
| **First Commit Date** | `2026-09-03 16:56:25 +0530` | `git log --reverse` (commit `3e5ef75`) |
| **Last Commit Date** | `2026-10-21 22:18:53 +0530` | `git log` (commit `fd191e4`) |
| **Total Commits** | `4` | `git rev-list --count HEAD` |
| **Active Development Span** | September 2026 – October 2026 (~7 weeks) | Git commit timestamps |
| **Team Structure** | **Solo Project** | `git shortlog -sn --all` (1 author: Smith Aherkar) |
| **Components Personally Built** | 100% of the codebase: Audio ingestion pipeline, dual-engine STT router, Map-Reduce summarization chains, structured extraction chains, ChromaDB RAG vector engine, Streamlit dark web interface, and terminal CLI. | Repository commit history (`app.py`, `core/`, `utils/`, `main.py`, `test.py`) |

---

## 4. Tech Stack

### Languages
- **Python 3.10+** (tested on Python 3.13): Primary programming language for all processing, chains, and interfaces.

### Backend and Orchestration
- **LangChain (`langchain`, `langchain-core`, `langchain-community`):** Orchestrates prompts, LLMs, and output parsers via LangChain Expression Language (LCEL).
- **LangChain MistralAI (`langchain-mistralai`):** Connects to Mistral AI chat endpoints (`ChatMistralAI`).
- **LangChain HuggingFace (`langchain-huggingface`) / `langchain-chroma`:** Bridges embeddings and Chroma vector storage into LangChain.
- **Requests (`requests`):** HTTP client for Sarvam AI REST API calls.
- **Python Dotenv (`python-dotenv`):** Environment variable loading from `.env`.

### Databases and Vector Stores
- **ChromaDB (`chromadb`, `langchain-chroma`):** In-process, local disk-persistent vector database used to index and retrieve transcript chunks.

### AI / ML Libraries and Models
- **Mistral AI (`mistral-small-latest`):** LLM used for text chunk summarization, final synthesis, title generation, action item extraction, decision extraction, question extraction, and RAG answering.
- **OpenAI Whisper (`openai-whisper`):** Local Transformer-based ASR (Automatic Speech Recognition) model used for offline English audio transcription (`whisper.load_model`).
- **Sarvam AI (`saaras:v2.5`):** Cloud STT & translation model for Indic/Hinglish speech via the `speech-to-text-translate` endpoint.
- **HuggingFace Embeddings (`sentence-transformers/all-MiniLM-L6-v2`):** 384-dimensional dense sentence embeddings running locally on CPU.
- **PyTorch (`torch`, `torchaudio`):** Backend compute runtime for Whisper and sentence-transformers.

### Media & Audio Processing
- **yt-dlp (`yt-dlp`):** Headless audio stream extraction from YouTube URLs.
- **pydub (`pydub`):** Audio slicing, channel conversion, and segment chunking.
- **audioop-lts (`audioop-lts`):** Drop-in `audioop` runtime for Python 3.13 compatibility with `pydub`.
- **FFmpeg (`ffmpeg-python` & system binary):** Audio transcoding, format extraction, and sample rate conversion.

### Frontend & UI
- **Streamlit (`streamlit`):** Full-stack reactive web interface with custom CSS styling (`Syne` and `JetBrains Mono` fonts, progress indicators, responsive cards, and live chat).

### Tools & Infrastructure
- **Git / GitHub:** Version control and source code hosting.
- **PowerShell / Bash:** Command-line environments for CLI execution and server administration.

---

## 5. Features

- **Universal Media Ingestion:** Paste any YouTube video URL or specify a local file path (`.mp4`, `.mkv`, `.mov`, `.mp3`, `.wav`) to automatically extract and convert the audio into standard 16kHz mono WAV chunks.
- **Dual-Engine Speech Routing:** Choose between English (processed privately and locally via Whisper) or Hinglish (transcribed and translated into English via Sarvam AI).
- **Automated Long-Audio Slicing:** Automatically splits large recordings into 10-minute segments for Whisper and 25-second micro-segments for Sarvam AI to avoid memory and API payload limits.
- **Executive Meeting Title Generation:** Synthesizes an executive, professional meeting title under 8 words from the transcript context.
- **Hierarchical Map-Reduce Summarization:** Summarizes lengthy transcripts chunk-by-chunk and reduces them into a unified, bullet-pointed executive brief without hitting LLM context window boundaries.
- **Action Item Extraction:** Extracts action items formatted as a numbered list with Task Description, Owner, and Deadline (or "Not specified").
- **Key Decision Cataloging:** Extracts all consensus decisions and strategic choices made during the discussion.
- **Open Questions Tracker:** Identifies unresolved blockers, open questions, and follow-up discussion points.
- **Context-Grounded Meeting Chat (RAG):** Interactive conversational assistant backed by ChromaDB and Mistral AI that answers specific user questions strictly using the transcript text.
- **Anti-Hallucination Guardrails:** Explicit prompt directives instruct the assistant to admit when information was not mentioned in the transcript rather than inventing facts.
- **Modern Cyberpunk Web UI:** Styled dark-theme interface with live step indicators for every pipeline stage and conversational chat history.
- **Interactive Terminal CLI:** Lightweight terminal runner (`main.py`) for headless servers or terminal users.

---

## 6. Architecture and Request Flow

### Request Flow

```
[User Input: YouTube URL or Local File Path]
       │
       ▼
[Audio Processor: utils/audio_processor.py]
  - If URL: yt-dlp extracts audio stream to WAV
  - If Local: pydub + FFmpeg normalizes to 16kHz Mono WAV
  - Slices audio into 10-minute chunks
       │
       ▼
[STT Router: core/transcriber.py]
  - If English: OpenAI Whisper loads model (default 'small') locally
  - If Hinglish: Audio chunk sliced into 25s pieces -> POST to api.sarvam.ai
  - Returns combined raw transcript text
       │
       ├──────────────────────────────────────────────┬───────────────────────────────┐
       ▼                                              ▼                               ▼
[Summarizer: core/summarizer.py]            [Extractor: core/extractor.py]    [Vector Store: core/vector_store.py]
  - Title: first 2,000 chars -> Mistral       - Action Items (Temp: 0.2)        - Text Splitter (500 chars, 50 overlap)
  - Map: 3,000-char chunks -> Mistral         - Key Decisions (Temp: 0.2)       - all-MiniLM-L6-v2 embeddings (CPU)
  - Reduce: Combined summaries -> Mistral     - Open Questions (Temp: 0.2)      - ChromaDB local collection: meeting_transcript
       │                                              │                               │
       └──────────────────────────────────────────────┴───────────────────────────────┘
                                                      │
                                                      ▼
                                         [RAG Engine: core/rag_engine.py]
                                           - User Query -> Similarity Search (Top k=4)
                                           - Formatted context + question -> Mistral AI
                                           - Strict negative constraint applied
                                                      │
                                                      ▼
                                       [Output Delivery: Streamlit UI / CLI]
                                         - Render Title, Summary, Transcript, Cards
                                         - Interactive Chat with conversation history
```

### Project Structure Tree

```
VidQuery/
├── .env.example              # Environment variables template (API keys, model sizes)
├── .gitignore                # Git ignore rules (.env, .venv, vector_db, downloads)
├── README.md                 # Project documentation and specifications
├── app.py                    # Streamlit web UI (CSS dark theme, status tracking, chat)
├── main.py                   # Terminal CLI runner and interactive REPL
├── requirements.txt          # Python library dependencies
├── test.py                   # Sequential end-to-end integration test script
│
├── core/                     # Core AI, transcription, and retrieval pipeline
│   ├── extractor.py          # Action items, key decisions, and open questions extraction
│   ├── rag_engine.py         # LCEL RAG prompt chain and QA invocation
│   ├── summarizer.py         # Map-Reduce transcript summarizer & title generator
│   ├── transcriber.py        # Dual STT router (Whisper local + Sarvam AI REST API)
│   └── vector_store.py       # ChromaDB vector store builder and retriever config
│
├── utils/                    # Audio acquisition and preprocessing
│   └── audio_processor.py    # yt-dlp downloader, 16kHz WAV normalization, 10m chunker
│
└── downloades/               # Default output directory for downloaded audio files
```

---

## 7. Data

| Attribute | Status / Value | Source File |
| :--- | :--- | :--- |
| **Tabular Dataset** | **NOT FOUND IN CODE** | No relational tables, CSVs, or tabular data files are used. |
| **SQL Queries / Joins** | **NOT FOUND IN CODE** | No SQL database or queries are present. |
| **Data Modality** | Unstructured Audio / Video / Natural Language Text | `utils/audio_processor.py`, `core/transcriber.py` |
| **Input Sources** | YouTube URLs or local video/audio files (`.mp4`, `.mkv`, `.mov`, `.mp3`, `.wav`) | `utils/audio_processor.py` (lines 8-36, 54-65) |
| **Target Variable / Labels** | **NOT FOUND IN CODE** | Unsupervised RAG, extraction, and generative summarization tasks; no supervised labels exist. |
| **Class Balance / Missing Values** | **NOT FOUND IN CODE** | Not applicable to this architecture. |
| **Audio Normalization Specs** | Sample rate: `16000` Hz, Channels: `1` (Mono), Codec: `PCM WAV` | `utils/audio_processor.py` (lines 16, 33) |
| **Audio Chunk Size** | `10` minutes (`600,000` ms) | `utils/audio_processor.py` (line 39) |
| **Sarvam Audio Slice Size** | `25` seconds (`25,000` ms) with 5-second safety margin under 30s limit | `core/transcriber.py` (lines 8, 72) |

---

## 8. Models and AI Components

### 1. Audio Transcription Models

#### A. OpenAI Whisper
- **Model Name:** `whisper` (`WHISPER_MODEL` environment variable, default: `small`).
  *(Source: `core/transcriber.py`, line 11)*
- **Execution Mode:** Local offline execution on device CPU/GPU via PyTorch.
  *(Source: `core/transcriber.py`, lines 27, 36)*
- **Task:** `"transcribe"` without translation.
  *(Source: `core/transcriber.py`, line 36)*
- **Why Chosen:** High transcription fidelity for English audio, runs locally with zero API cost, completely private.
- **Alternatives Rejected / Noted in Code:** Noted in code: `tiny`, `base`, `small`, `medium`, `large` model sizes supported via environment configuration.
  *(Source: `README.md` lines 213-214, `.env.example` line 13)*

#### B. Sarvam AI STT & Translation
- **Model Name:** `saaras:v2.5` (`SARVAM_STT_MODEL` environment variable, default: `saaras:v2.5`).
  *(Source: `core/transcriber.py`, line 16)*
- **Endpoint:** `https://api.sarvam.ai/speech-to-text-translate`
  *(Source: `core/transcriber.py`, line 15)*
- **Parameters:** `{"model": "saaras:v2.5", "with_diarization": "false"}`
  *(Source: `core/transcriber.py`, line 46)*
- **Why Chosen:** Specifically optimized for Indian English accents and code-mixed Hindi-English ("Hinglish"), translating directly to English during speech recognition.

---

### 2. Large Language Model (LLM) — Mistral AI

- **Model Identifier:** `mistral-small-latest`
  *(Source: `core/summarizer.py` line 10, `core/extractor.py` line 11, `core/rag_engine.py` line 10)*
- **Hyperparameters:**
  - `temperature = 0.3` for Map-Reduce summarization and RAG Q&A.
    *(Source: `core/summarizer.py` line 10, `core/rag_engine.py` line 12)*
  - `temperature = 0.2` for structured information extraction (Action Items, Decisions, Questions).
    *(Source: `core/extractor.py` line 11)*
- **Why Chosen:** High instruction-following capability, low latency, and cost-effective API pricing relative to large frontier models.

---

### 3. Text Splitting, Embeddings & RAG Configuration

| Parameter | Value | Source File |
| :--- | :--- | :--- |
| **Summarizer Chunk Size** | `3000` characters | `core/summarizer.py` (line 15) |
| **Summarizer Chunk Overlap** | `200` characters | `core/summarizer.py` (line 16) |
| **Title Input Truncation** | First `2000` characters | `core/summarizer.py` (line 75) |
| **RAG Vector Chunk Size** | `500` characters | `core/vector_store.py` (line 21) |
| **RAG Vector Chunk Overlap** | `50` characters | `core/vector_store.py` (line 22) |
| **Embedding Model** | `all-MiniLM-L6-v2` (`sentence-transformers`) | `core/vector_store.py` (line 9) |
| **Embedding Device** | `"cpu"` | `core/vector_store.py` (line 14) |
| **Vector Store** | ChromaDB (`langchain-chroma`) | `core/vector_store.py` (lines 2, 32-37) |
| **Chroma Collection Name** | `"meeting_transcript"` | `core/vector_store.py` (line 8) |
| **Chroma Persistence Directory** | `"vector_db"` | `core/vector_store.py` (line 7) |
| **Retrieval Strategy** | Similarity Search | `core/vector_store.py` (line 55) |
| **Retriever Top-k** | `4` documents | `core/vector_store.py` (line 53) & `core/rag_engine.py` (line 22) |

---

### 4. Metrics & Evaluation
- **Classification / Regression Metrics (Accuracy, F1, R2, RMSE):** **NOT FOUND IN CODE**. The repository does not include an offline validation split or benchmark evaluation script computing statistical metrics.
- **Performance Evaluation Note:** The prompts rely on zero-shot generation and in-context retrieval. Grounding is strictly constrained via prompt engineering directives rather than an automated RAG triad benchmark (e.g., Ragas/TruLens).

---

## 9. Backend

VidQuery is designed as a modular local Python service rather than a standalone REST server (e.g. FastAPI/Flask). Modules are called in-process by `app.py` or `main.py`.

### External API Endpoints Called

| Method | Endpoint URL | Purpose | Request Body / Payload | Response Body | Source File |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `POST` | `https://api.sarvam.ai/speech-to-text-translate` | Translate and transcribe $\le$30s audio pieces | Multipart Form Data: `file` (WAV audio), `model` (`saaras:v2.5`), `with_diarization` (`"false"`). Header: `api-subscription-key` | JSON: `{"transcript": "..."}` | `core/transcriber.py` (lines 15, 45-60) |
| `POST` | Mistral AI Chat Completions endpoint (via SDK) | Generate summaries, extractions, titles, and Q&A answers | Prompt messages payload, model identifier, temperature | Chat completion chunk / string | `core/summarizer.py`, `core/extractor.py`, `core/rag_engine.py` |

### Internal Functional API (Python Modules)

| Module / Function | Inputs | Outputs | Purpose | Source File |
| :--- | :--- | :--- | :--- | :--- |
| `process_input(source: str)` | Media path or YouTube URL | `list[str]` (file paths to WAV chunks) | Converts media to 16kHz mono WAV chunks of 10 min | `utils/audio_processor.py` (lines 54-65) |
| `transcribe_all(chunks, language)` | `chunks: list`, `language: str` | `str` (consolidated transcript) | Routes chunks to Whisper or Sarvam and concatenates | `core/transcriber.py` (lines 106-123) |
| `generate_title(transcript: str)` | `transcript: str` | `str` (meeting title $\le$ 8 words) | Uses Mistral AI to create concise meeting title | `core/summarizer.py` (lines 56-75) |
| `summarize(transcript: str)` | `transcript: str` | `str` (bulleted executive summary) | Map-Reduce summarization chain over chunks | `core/summarizer.py` (lines 21-54) |
| `extract_action_items(transcript)` | `transcript: str` | `str` (numbered action items) | Extracts task, owner, deadline from transcript | `core/extractor.py` (lines 24-35) |
| `extract_key_decisions(transcript)` | `transcript: str` | `str` (numbered decisions) | Extracts agreed consensus decisions | `core/extractor.py` (lines 37-44) |
| `extract_questions(transcript)` | `transcript: str` | `str` (numbered questions) | Extracts unresolved queries needing follow-up | `core/extractor.py` (lines 46-52) |
| `build_rag_chain(transcript: str)` | `transcript: str` | LCEL Runnable chain | Builds ChromaDB collection and returns RAG chain | `core/rag_engine.py` (lines 18-56) |
| `ask_question(rag_chain, question)`| `rag_chain`, `question: str` | `str` (grounded answer) | Invokes RAG chain with context and question | `core/rag_engine.py` (lines 93-97) |

### Validation & Error Handling
- **Input Validation:** Checks if the input source is empty or blank before processing (`app.py`, line 365).
- **Audio Clean-Up:** Sliced 25-second temporary files generated for Sarvam AI are wrapped in a `try...finally` block to guarantee file deletion even on network failure (`core/transcriber.py`, lines 82-88).
- **Missing API Keys:** Explicit runtime error raised if `SARVAM_API_KEY` is missing when Hinglish mode is selected (`core/transcriber.py`, lines 68-69).
- **HTTP Errors:** Calls `response.raise_for_status()` when Sarvam AI returns non-200 status codes and prints error bodies (`core/transcriber.py`, lines 55-58).
- **Authentication:** None for local application access; API key authentication (`MISTRAL_API_KEY`, `SARVAM_API_KEY`) used for external services.

---

## 10. Database

- **Relational Tables:** **NOT FOUND IN CODE** (No SQL/Relational database is used).
- **Vector Database (ChromaDB):**
  - **Storage Type:** Local disk persistence (`vector_db/` folder).
    *(Source: `core/vector_store.py`, line 7)*
  - **Collection:** `meeting_transcript`
    *(Source: `core/vector_store.py`, line 8)*
  - **Document Schema:**

| Field / Attribute | Data Type | Description | Source File |
| :--- | :--- | :--- | :--- |
| `page_content` | `str` | 500-character split text segment from meeting transcript | `core/vector_store.py` (lines 24-28) |
| `metadata.chunk_index` | `int` | Sequential chunk index integer (`0, 1, 2, ...`) | `core/vector_store.py` (line 27) |
| `embedding` | `vector[384]` | 384-dimensional dense float vector from `all-MiniLM-L6-v2` | `core/vector_store.py` (lines 9, 31-36) |

---

## 11. Frontend

The frontend is implemented in **Streamlit** ([`app.py`](file:///d:/Vidquery/app.py)) spanning 545 lines of code.

### Layout and Visual Components
- **Custom Theme:** Cyberpunk-inspired dark interface with custom CSS styling using Google Fonts (`Syne` for titles, `JetBrains Mono` for monospace labels and body text).
- **Background:** CSS linear gradient animated background grid pattern (`app.py`, lines 52-65).
- **Sidebar Panel:**
  - Input field for YouTube URL or local file path.
  - Language selection dropdown (`english` or `hinglish`).
  - **"⚡ Analyse"** action trigger button.
  - **Pipeline Status Tracker:** Live indicator bar with pulsing dots (`dot-active`, `dot-done`, `dot-pending`) tracking 6 sequential stages:
    1. 🔊 Audio Processing (`audio`)
    2. 📝 Transcription (`transcript`)
    3. 🏷️ Title Generation (`title`)
    4. 📋 Summarisation (`summary`)
    5. 🔍 Extraction (`extract`)
    6. 🧠 RAG Engine (`rag`)
- **Main Dashboard (Results View):**
  - **Session Title Banner:** Clean card displaying the title generated by Mistral AI.
  - **Two-Column Top Grid:**
    - Left Column (3/5 width): Executive summary card in bullet points.
    - Right Column (2/5 width): Expandable collapsible transcript viewer (`max-height: 300px`).
  - **Three-Column Bottom Grid:**
    - Card 1: ✅ Action Items (tasks, owners, deadlines).
    - Card 2: 🔑 Key Decisions.
    - Card 3: ❓ Open Questions.
  - **Interactive RAG Chat Section:**
    - Visual chat history container with color-coded chat bubbles (User in purple, Assistant in cyan).
    - Input text box and "Send →" button.
    - "🗑️ Clear Chat" button to reset `st.session_state.chat_history`.
- **Empty State:** Illustrated welcome card when no video has been analyzed yet.

---

## 12. Setup and Usage

### Prerequisites
1. **Python 3.10+** (tested and verified on Python 3.13).
2. **FFmpeg binary** installed and accessible on your system `PATH`.
   - Windows: `winget install "FFmpeg (Essentials Build)"` or `choco install ffmpeg`
   - macOS: `brew install ffmpeg`
   - Linux: `sudo apt install ffmpeg`
3. **API Keys:**
   - Mistral AI API Key (required for summarization, extractions, and RAG).
   - Sarvam AI API Key (optional, required only for Hinglish mode).

### Installation Steps

1. **Clone the repository:**
   ```bash
   git clone https://github.com/smithaherkar/VidQuery.git
   cd VidQuery
   ```

2. **Create and activate a virtual environment:**
   ```bash
   # Windows PowerShell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1

   # Linux / macOS
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. **Install dependencies:**
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   pip install audioop-lts langchain-chroma
   ```

4. **Configure environment variables:**
   Create a `.env` file from `.env.example`:
   ```bash
   # Windows
   Copy-Item .env.example .env

   # Linux / macOS
   cp .env.example .env
   ```

   Populate your `.env` file:
   ```env
   # Required
   MISTRAL_API_KEY=your_mistral_api_key_here

   # Optional (Required only for Hinglish)
   SARVAM_API_KEY=your_sarvam_api_key_here

   # Model configurations
   WHISPER_MODEL=small
   SARVAM_STT_MODEL=saaras:v2.5
   ```

### Running the Application

- **Option A: Streamlit Web Interface (Recommended)**
  ```bash
  streamlit run app.py
  ```
  Open `http://localhost:8501` in your browser.

- **Option B: Terminal CLI Mode**
  ```bash
  python main.py
  ```

- **Option C: Automated Integration Test**
  ```bash
  python test.py
  ```

---

## 13. Challenges and Decisions

### 1. Handling Long Multi-Hour Media Without Memory Crashes
- **Problem:** Feeding a 2-hour high-bitrate video directly into memory causes FFmpeg memory overflow and crashes Whisper.
- **Decision & Solution:** Standardized all audio into single-channel (mono) 16kHz WAV format and introduced an automated chunker (`chunk_audio`) that segments audio into 10-minute intervals (`utils/audio_processor.py`, lines 39-52).

### 2. Sarvam AI 30-Second API Payload Ceiling
- **Problem:** Sarvam AI’s synchronous speech translation endpoint (`api.sarvam.ai/speech-to-text-translate`) rejects audio files longer than 30 seconds with an HTTP error.
- **Decision & Solution:** Implemented sub-chunking inside `core/transcriber.py` (lines 63-90). The 10-minute chunk is sliced into 25-second micro-segments (5-second safety margin), uploaded sequentially, aggregated into text, and temporary WAV files are removed via a `finally` block.

### 3. Python 3.13 `audioop` Deprecation
- **Problem:** Python 3.13 removed the internal `audioop` C extension module (PEP 594), causing `pydub` imports to fail with `ModuleNotFoundError: No module named 'pyaudioop'`.
- **Decision & Solution:** Integrated the standalone `audioop-lts` package into the environment, allowing `pydub` to operate seamlessly on modern Python 3.13 runtimes without requiring a Python version downgrade.

### 4. Overcoming LLM Context Limits and Attention Loss (Map-Reduce)
- **Problem:** Passing 25,000+ words of meeting transcript into a single LLM prompt leads to attention dilution ("Lost in the Middle" syndrome) and truncation.
- **Decision & Solution:** Implemented a two-stage Map-Reduce summarization architecture in `core/summarizer.py`:
  - **Map:** Split transcript into 3,000-character chunks (200-character overlap) and summarize each portion independently.
  - **Reduce:** Concatenate intermediate summaries and prompt Mistral AI to synthesize a unified executive brief.

### 5. Grounding Meeting Chat & Eliminating RAG Hallucinations
- **Problem:** LLMs naturally hallucinate general external information when a user asks about topics not covered in the meeting.
- **Decision & Solution:** Constructed high-density 500-character chunks with strict negative prompt guardrails (`core/rag_engine.py`, lines 30-36):
  > *"Answer the user's question based ONLY on the meeting transcript context provided below. If the answer is not found in the context, say: 'I could not find this information in the meeting transcript.'"*

---

## 14. Limitations and Future Work

### Limitations
1. **Synchronous Execution:** Long videos process synchronously. In Streamlit, processing runs inside the request cycle; a page refresh interrupts execution.
2. **Sequential API Calls for Sarvam:** Sarvam's 25-second audio micro-pieces are currently uploaded sequentially via `requests.post` rather than in parallel asynchronously (`aiohttp`), which increases transcription time on long Hinglish meetings.
3. **No Speaker Diarization:** Transcripts do not yet label who spoke each line (e.g. *Speaker 1 vs Speaker 2*), so action items rely on speaker references made within the conversation text.
4. **No Formal Metric Benchmarks:** The repository contains no automated accuracy/F1/ROUGE benchmark evaluation suite.

### Next Improvements (Future Roadmap)
- [ ] **Asynchronous Background Workers:** Decouple media processing using Celery and Redis to enable background job processing with WebSocket updates.
- [ ] **Speaker Diarization:** Integrate `pyannote.audio` to identify and label individual speakers with timestamps.
- [ ] **Hybrid Search (BM25 + Dense Vectors):** Combine vector similarity with lexical keyword search (Reciprocal Rank Fusion) for better acronym and name retrieval.
- [ ] **Parallel Micro-Chunking:** Use `asyncio` and `aiohttp` to dispatch Sarvam STT chunks concurrently.
- [ ] **Export to PDF/Markdown:** Add one-click export buttons in the UI using `reportlab` or `fpdf2` (already present in `requirements.txt`).

---

## 15. PROJECT FACTS (Verified Facts Table)

*Every row is strictly verified against codebase files. Where no data exists in code, it is marked as NOT FOUND IN CODE.*

| Item | Value | Source File |
| :--- | :--- | :--- |
| **Project Name** | `VidQuery` | `README.md` (line 3) |
| **GitHub Repository Link** | `https://github.com/smithaherkar/VidQuery.git` | `git remote -v` |
| **Live Demo Link** | **NOT FOUND IN CODE** | No public cloud URL found in codebase |
| **Start and End Month/Year** | September 2026 – October 2026 | Git commits `3e5ef75` (2026-09-03) to `fd191e4` (2026-10-21) |
| **Solo or Team, and Role** | Solo project (100% written by Smith Aherkar) | `git shortlog -sn --all` |
| **Number of Internal API Endpoints** | `0` (In-process modular Python service, no FastAPI/Flask router) | Project directory inspection |
| **Number of External API Endpoints Called** | `2` (`POST https://api.sarvam.ai/speech-to-text-translate` and Mistral AI Chat API) | `core/transcriber.py` (line 15), `core/summarizer.py` (line 10) |
| **Number of Database Tables** | `0` relational tables (`1` ChromaDB collection named `meeting_transcript`) | `core/vector_store.py` (line 8) |
| **Number of Pipeline Steps (UI)** | `6` (`audio`, `transcript`, `title`, `summary`, `extract`, `rag`) | `app.py` (lines 349-355) |
| **Number of AI Models Used** | `4` (`Whisper-small`, `Sarvam saaras:v2.5`, `Mistral-small-latest`, `all-MiniLM-L6-v2`) | `core/transcriber.py`, `core/summarizer.py`, `core/vector_store.py` |
| **Dataset Size (Rows)** | **NOT FOUND IN CODE** (Unstructured audio/video input; no row-based dataset) | Workspace inspection |
| **Model Metrics (Train / Test)** | **NOT FOUND IN CODE** (No quantitative benchmark metrics computed in code) | Workspace inspection |
| **Number of Input Features** | **NOT FOUND IN CODE** (Not a feature-engineered tabular ML model) | Workspace inspection |
| **Performance Figures (Latency, Accuracy)** | **NOT FOUND IN CODE** (No benchmark runtime benchmarks logged in code) | Workspace inspection |
| **Number of Test Files** | `1` integration test script (`test.py`) with `0` formal `pytest`/`unittest` test suites | `test.py` |
| **Supported Input Formats** | YouTube URLs, local audio/video (`.mp4`, `.mkv`, `.mov`, `.mp3`, `.wav`) | `utils/audio_processor.py` (lines 8, 29) |
| **Supported Languages** | `2` (`english`, `hinglish`) | `app.py` (line 341), `main.py` (line 43) |
| **Number of App Outputs** | `6` (`title`, `transcript`, `summary`, `action_items`, `key_decisions`, `open_questions`) | `app.py` (lines 408-416), `main.py` (lines 30-38) |
| **Deployment Status** | Local only (Streamlit server on `localhost:8501`, no Dockerfile found in code) | `app.py`, repository inspection |

---

## 16. ATS KEYWORDS

1. **Retrieval-Augmented Generation (RAG)**
2. **LangChain (LCEL)**
3. **OpenAI Whisper**
4. **Speech-to-Text (STT)**
5. **ChromaDB**
6. **Sentence Transformers**
7. **Mistral AI (`mistral-small`)**
8. **Map-Reduce Summarization**
9. **Vector Embeddings (`all-MiniLM-L6-v2`)**
10. **Audio Preprocessing & Normalization**
11. **FFmpeg**
12. **pydub**
13. **yt-dlp**
14. **Streamlit**
15. **Indic NLP & Hinglish Translation**
16. **Information Extraction (Action Items, Decisions)**
17. **Semantic Search & Document Chunking**
18. **Prompt Engineering & Anti-Hallucination Guardrails**
19. **Python 3.13**
20. **REST API Integration**

---

## 17. RESUME CLAIM CHECK

Compare your resume bullets against the verified codebase facts below. Paste your draft bullets into this section to verify their claim status.

| Claim Status | Criteria |
| :--- | :--- |
| **SUPPORTED** | Claim matches exact implementation, file, and verified number in code. |
| **PARTLY SUPPORTED** | Claim describes real architecture, but numbers/scope are estimated or not measured in code. |
| **NOT FOUND** | Technology, metric, or feature is absent from the repository. |

### Reference Evaluation of Standard Claims for this Repo:

1. **"Architected an end-to-end meeting intelligence pipeline using Whisper, Sarvam AI, and Mistral AI over LangChain LCEL."**
   - **Status:** **SUPPORTED**
   - **Source:** [`core/transcriber.py`](file:///d:/Vidquery/core/transcriber.py), [`core/summarizer.py`](file:///d:/Vidquery/core/summarizer.py), [`core/rag_engine.py`](file:///d:/Vidquery/core/rag_engine.py).

2. **"Engineered a Map-Reduce summarization engine chunking audio transcripts into 3,000-character segments to overcome LLM context limits."**
   - **Status:** **SUPPORTED**
   - **Source:** [`core/summarizer.py`](file:///d:/Vidquery/core/summarizer.py) (lines 14-35).

3. **"Built local vector database with ChromaDB and all-MiniLM-L6-v2 embeddings retrieving top-4 chunks for grounded RAG Q&A."**
   - **Status:** **SUPPORTED**
   - **Source:** [`core/vector_store.py`](file:///d:/Vidquery/core/vector_store.py) (lines 7-57), [`core/rag_engine.py`](file:///d:/Vidquery/core/rag_engine.py) (line 22).

4. **"Achieved 95% summarization accuracy and reduced review time by 80%."**
   - **Status:** **NOT FOUND** (or PARTLY SUPPORTED if rephrased)
   - **Correction:** Quantitative percentage gains (95%, 80%) are **NOT FOUND IN CODE** (no evaluation benchmark or latency logging in repository). Phrase instead as: *"Engineered automated Map-Reduce summarization and structured extraction pipeline, replacing manual video review with instant executive briefs."*

5. **"Deployed containerized Docker microservice on AWS ECS with PostgreSQL."**
   - **Status:** **NOT FOUND**
   - **Correction:** No `Dockerfile`, `docker-compose.yml`, or PostgreSQL database exists in the repository. The project runs locally with Streamlit and ChromaDB.

---

```
[PASTE CURRENT BULLETS HERE]
```
