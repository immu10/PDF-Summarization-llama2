> This was a college course project (NLP).

# DOCUMENT SUMMARIZATION

## Overview
A Streamlit app that summarizes documents using a locally running **Mistral** model served through the **Ollama** backend. You upload a PDF, DOCX, or TXT file, optionally steer the output with a custom prompt, and choose how condensed the summary should be. Alongside summarization, the app can explain words or sentences in plain language, look up dictionary definitions, and read text aloud via text-to-speech.

Everything runs locally — no API keys or cloud calls. We initially aimed for LLaMA 3.3 but it exceeded local hardware limits (≈33.7 GiB RAM required), so Mistral is used instead.

### Features
- Upload a **PDF**, **DOCX**, or **TXT** file and get a summary.
- Custom prompt box — ask a question or steer the summary instead of just "summarize this".
- Adjustable summary length via a slider (lower = more detailed explanation, higher = more condensed summary).
- **Explain** a word or sentence in plain language (via Mistral).
- **Lookup** a single word's dictionary definitions (via WordNet).
- **Listen** — text-to-speech playback of the input and the generated summary (saved as MP3).

## Architecture
The app is split into a UI layer and a processing layer, with the LLM served separately by Ollama.

```
        ┌─────────────────┐
        │   tres_uno.py   │   Streamlit UI
        │  (upload, prompt│   - file upload (pdf/docx/txt)
        │  slider, buttons│   - prompt box + length slider
        │   audio output) │   - explain / lookup / listen
        └────────┬────────┘
                 │ calls
                 ▼
        ┌─────────────────┐
        │     tres.py     │   Processing + LLM logic
        │                 │   - file_preprocessing(): load & chunk
        │                 │   - llm_pipeline(): summarize per chunk
        │                 │   - explain_text(): Mistral explanation
        │                 │   - explain_with_wordnet(): definitions
        │                 │   - generate_tts(): text-to-speech
        └───┬─────────┬───┘
            │         │
   LangChain│         │ ollama.generate
 loaders &  │         ▼
 splitters  │   ┌───────────┐
            │   │  Ollama   │  local Mistral model
            │   └───────────┘
            ▼
   PDF / DOCX / TXT files
```

**Flow:** a file is loaded and split into chunks (LangChain `RecursiveCharacterTextSplitter`), each chunk is sent to Mistral via Ollama with the user's prompt, and the per-chunk responses are joined into the final summary. The summary is then optionally converted to speech. The chunk size, overlap, and token budget all scale off the length slider.

## Tech Stack
| Layer | Tools |
|-------|-------|
| UI | Streamlit |
| LLM | Mistral via Ollama (local) |
| Document loading / chunking | LangChain (`PyPDFLoader`, `RecursiveCharacterTextSplitter`), python-docx, pypandoc |
| Dictionary lookup | NLTK WordNet |
| Text-to-speech | pyttsx3 / gTTS |
| Evaluation | rouge-score |
| Language | Python |

## Setup
1. Install [Ollama](https://ollama.com/) and pull the model:
   ```bash
   ollama pull mistral
   ```
2. Install Python dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. If `pypandoc` complains about a missing Pandoc binary, run once in Python:
   ```python
   import pypandoc
   pypandoc.download_pandoc()
   ```
4. NLTK WordNet data is needed for the Lookup feature:
   ```python
   import nltk
   nltk.download('wordnet')
   ```

## Running
```bash
streamlit run tres_uno.py
```

## Project Files
- [tres_uno.py](tres_uno.py) — Streamlit UI (main entry point).
- [tres.py](tres.py) — processing and LLM logic.
- [dos.py](dos.py) — earlier PDF-only version using a HuggingFace `transformers` pipeline instead of Ollama (kept for reference).
- [query_data.py](query_data.py) / [uno.py](uno.py) — experimental RAG-style retrieval over a folder of PDFs using Chroma + Ollama (separate from the main app).
- [eval.py](eval.py) — standalone ROUGE-score evaluation snippet for summary quality.

## Notes
This is coursework, not production software — expect rough edges. Feel free to extend it.
