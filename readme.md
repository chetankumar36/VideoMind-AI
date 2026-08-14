# 🎬 VideoMind AI

### AI-Powered Video & Meeting Intelligence Assistant

VideoMind AI is an AI-powered video and meeting assistant built with Python. It transforms YouTube videos and uploaded audio/video files into structured, searchable knowledge.

The application can extract audio, transcribe speech locally using Whisper, translate Hinglish/Hindi speech into English using Sarvam AI, summarize the content using Mistral, extract action items, key decisions and open questions, and provide an interactive RAG-based chat interface for asking questions about the processed meeting.

Built with **Python, Streamlit, Whisper, Sarvam AI, LangChain, Mistral, ChromaDB and HuggingFace embeddings**.

---

## Table of Contents

- [Features](#-features)
- [System Architecture](#️-system-architecture)
- [Project Structure](#-project-structure)
- [File Responsibilities](#-file-responsibilities)
- [Technology Stack](#️-technology-stack)
- [Requirements](#️-requirements)
- [Installation](#-installation)
- [Environment Variables](#-environment-variables)
- [Running VideoMind AI](#️-running-videomind-ai)
- [Running the CLI Pipeline](#-running-the-cli-pipeline)
- [Testing](#-testing)
- [Configuration Reference](#-configuration-reference)
- [Generated Files](#-generated-files)
- [Troubleshooting](#-troubleshooting)
- [Why RAG?](#-why-rag)
- [Future Improvements](#-future-improvements)
- [Project Status](#-current-project-status)
- [License](#-license)
- [Acknowledgements](#-acknowledgements)

---

## ✨ Features

### 🎥 YouTube Video Processing

Provide a YouTube URL and VideoMind AI will:

1. Download the best available audio using `yt-dlp`
2. Convert the audio to WAV
3. Convert audio to mono 16 kHz when processing uploaded files
4. Split long audio into manageable chunks
5. Send the chunks through the selected transcription pipeline

### 📁 Local File Processing

VideoMind AI can also process local audio/video files. Supported input depends on the FFmpeg/PyDub codecs available on the system. Typical formats include:

- WAV
- MP3
- MP4
- M4A
- WebM
- Other FFmpeg-supported formats

### 🎙️ Local Whisper Transcription

For English transcription, VideoMind AI uses OpenAI Whisper locally.

```
Audio → Whisper → English Transcript
```

The Whisper model can be configured using:

```env
WHISPER_MODEL=small
```

Available Whisper model choices include: `tiny`, `base`, `small`, `medium`, `large`.

> Larger models generally provide better transcription quality but require more RAM/CPU/GPU resources.

### 🌐 Hinglish / Hindi → English

For Hinglish transcription and translation, VideoMind AI uses Sarvam AI's Speech-to-Text Translate API. The application sends short audio pieces to the API and receives an English transcript.

The current implementation uses `saaras:v2.5`. Audio is split into 25-second pieces before being sent to Sarvam because the synchronous API has a 30-second audio limitation.

```
Hinglish / Hindi Audio → 25-second audio pieces → Sarvam AI → English Transcript
```

### 🧠 AI Meeting Analysis

After transcription, the transcript is processed using Mistral (`mistral-small-latest`) through LangChain for:

- Meeting title generation
- Meeting summarization
- Action-item extraction
- Key-decision extraction
- Open-question extraction
- RAG question answering

### 📋 Automatic Summary

VideoMind AI generates a professional meeting summary from the transcript. Long transcripts are split into smaller sections before summarization.

```
Full Transcript → Transcript Chunks → Individual Summaries → Combined Summary → Final Meeting Summary
```

This map-and-combine approach allows the application to handle transcripts that are too large to send to the LLM in one request.

### 🏷️ Automatic Meeting Title

The application generates a short professional title (≈8 words) from the transcript, e.g. *"Quarterly Product Planning Meeting."*

### ✅ Action Item Extraction

VideoMind AI extracts actionable tasks from the meeting. For each action item it attempts to identify a **task description**, **owner**, and **deadline**.

```
1. Prepare the project report
   Owner: Rahul
   Deadline: Friday

2. Contact the client
   Owner: Priya
   Deadline: Not specified
```

### 🔑 Key Decision Extraction

The system identifies important decisions made during the meeting.

```
1. The project deadline was moved to March 15.
2. The team decided to use PostgreSQL.
3. The next client meeting will be held on Monday.
```

### ❓ Open Questions

VideoMind AI also extracts unresolved questions and topics requiring follow-up.

```
1. Who will handle the production deployment?
2. What is the final budget?
3. When will the client provide the required assets?
```

### 🔎 RAG — Chat With Your Meeting

One of the main features of VideoMind AI is the ability to chat with the processed meeting transcript using a Retrieval-Augmented Generation (RAG) pipeline.

**RAG Architecture**

```
Transcript → Text Splitting → Embeddings → ChromaDB → Similarity Search
→ Relevant Transcript Chunks → Mistral → Answer
```

### 🗄️ Vector Database

VideoMind AI uses **ChromaDB** as the local vector database. The transcript is split into chunks using `RecursiveCharacterTextSplitter`.

| Setting | Value |
|---|---|
| Chunk size | 500 characters |
| Chunk overlap | 50 characters |
| Embedding model | `all-MiniLM-L6-v2` (HuggingFace) |
| Storage path | `vector_db/` |

> The `vector_db/` directory is intentionally excluded from Git because it is generated application data.

### 💬 Ask Questions About the Meeting

After processing a video, users can ask questions such as:

- What were the main topics discussed?
- What did Rahul agree to do?
- What were the key decisions?
- Who is responsible for the deployment?
- What is the project deadline?
- What problems were discussed?
- What are the next steps?
- Was a budget discussed?

The RAG system retrieves relevant transcript sections and sends them to Mistral. The model is instructed to answer only from the retrieved meeting context. If the information cannot be found, it responds:

> *"I could not find this information in the meeting transcript."*

### 🔗 LangChain LCEL

VideoMind AI uses LangChain Expression Language (LCEL) to build the LLM and RAG pipelines.

```
Question → Retriever → Relevant Documents → Prompt → Mistral → Answer
```

Key LangChain components used:

- `ChatPromptTemplate`
- `StrOutputParser`
- `RunnablePassthrough`
- `RunnableLambda`
- Chroma retrievers
- Mistral chat models

### 🖥️ Streamlit Interface

The UI is built using Streamlit and includes:

- Video/URL input
- Language selection
- Processing status
- Transcript display
- AI-generated title
- Meeting summary
- Action items
- Key decisions
- Open questions
- RAG chat
- Export functionality

The interface uses a custom dark-themed design with animated UI elements and cards.

---

## 🏗️ System Architecture

```
                         VideoMind AI
                              │
             ┌────────────────┴────────────────┐
             │                                  │
        YouTube URL                        Local File
             │                                  │
             ▼                                  ▼
          yt-dlp                             PyDub
             │                                  │
             └──────────────┬───────────────────┘
                             ▼
                        Audio / WAV
                             │
                             ▼
                      Audio Chunking
                             │
                 ┌───────────┴───────────┐
                 │                       │
              English                Hinglish
                 │                       │
                 ▼                       ▼
              Whisper                Sarvam AI
                 │                       │
                 └───────────┬───────────┘
                             ▼
                        Transcript
                             │
             ┌───────────────┼───────────────┐
             │                │               │
             ▼                ▼               ▼
          Summary         Extraction         RAG
                              │                │
                    ┌─────────┼─────────┐      │
                    │         │         │      │
                 Actions  Decisions  Questions  │
                                                 ▼
                                            Embeddings
                                                 │
                                                 ▼
                                             ChromaDB
                                                 │
                                                 ▼
                                        Similarity Search
                             │                   │
                             └─────────┬─────────┘
                                       ▼
                                    Mistral
                                       │
                                       ▼
                                 Streamlit UI
                                       │
                                       ▼
                          User / Meeting Insights
```

---

## 📂 Project Structure

```
VideoMind-AI/
│
├── app.py
├── main.py
├── test.py
├── requirements.txt
├── README.md
├── .gitignore
│
├── core/
│   ├── extractor.py
│   ├── rag_engine.py
│   ├── summarizer.py
│   ├── transcriber.py
│   └── vector_store.py
│
└── utils/
    └── audio_processor.py
```

---

## 📌 File Responsibilities

### `app.py`

Main Streamlit user interface. Responsible for:

- UI
- Input handling
- Processing workflow
- Displaying transcript and AI results
- RAG chat
- User interaction

### `main.py`

CLI/application pipeline entry point.

```
source
   ↓
process_input()
   ↓
transcribe_all()
   ↓
generate_title()
   ↓
summarize()
   ↓
extract_action_items()
   ↓
extract_key_decisions()
   ↓
extract_questions()
   ↓
build_rag_chain()
```

### `utils/audio_processor.py`

Responsible for YouTube audio downloading, audio conversion, WAV generation, audio normalization, and audio chunking.

Main functions: `download_youtube_audio()`, `convert_to_wav()`, `chunk_audio()`, `process_input()`

### `core/transcriber.py`

Responsible for speech-to-text. Supports English via Whisper and Hinglish via Sarvam AI.

Main functions: `load_model()`, `transcribe_chunk_whisper()`, `transcribe_chunk_sarvam()`, `transcribe_chunk()`, `transcribe_all()`

### `core/summarizer.py`

Responsible for transcript splitting, partial summaries, final summary generation, and meeting title generation. Uses LangChain, Mistral, and LCEL.

### `core/extractor.py`

Responsible for extracting action items, key decisions, and open questions.

### `core/vector_store.py`

Responsible for creating embeddings, splitting transcript text, creating ChromaDB collections, persisting vectors, and creating retrievers.

Embedding model: `all-MiniLM-L6-v2`

### `core/rag_engine.py`

Responsible for the RAG pipeline.

Main functions: `build_rag_chain()`, `load_rag_chain()`, `ask_question()`

---

## 🛠️ Technology Stack

| Component | Technology |
|---|---|
| Programming Language | Python |
| UI | Streamlit |
| Video Download | yt-dlp |
| Audio Processing | PyDub |
| Audio Backend | FFmpeg |
| Local STT | OpenAI Whisper |
| Hindi/Hinglish Translation | Sarvam AI |
| LLM | Mistral |
| LLM Framework | LangChain |
| RAG | LangChain LCEL |
| Vector Database | ChromaDB |
| Embeddings | HuggingFace Sentence Transformers |
| Environment Variables | python-dotenv |
| HTTP Requests | Requests |
| PDF Support | ReportLab / FPDF2 |

---

## ⚙️ Requirements

Recommended: **Python 3.10+**

Project dependencies are listed in `requirements.txt`.

---

## 🚀 Installation

### 1. Clone the repository

```bash
git clone https://github.com/chetankumar36/VideoMind-AI.git
cd VideoMind-AI
```

### 2. Create a virtual environment (Windows)

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

You should see `(.venv)` or your environment name in the terminal.

### 3. Install dependencies

Using pip:

```bash
pip install -r requirements.txt
```

Using uv:

```bash
uv pip install -r requirements.txt
```

### 4. Install FFmpeg

FFmpeg is required for audio/video processing. Check whether it's installed:

```bash
ffmpeg -version
```

If it is not installed, install FFmpeg and make sure the executable is available on your system PATH. VideoMind AI uses FFmpeg indirectly through PyDub and ffmpeg-python.

---

## 🔐 Environment Variables

Create a `.env` file in the project root:

```env
MISTRAL_API_KEY=your_mistral_api_key
SARVAM_API_KEY=your_sarvam_api_key
WHISPER_MODEL=small
SARVAM_STT_MODEL=saaras:v2.5
```

**Required API keys:**

| Use case | Variable | Notes |
|---|---|---|
| English/local transcription | `WHISPER_MODEL` | No API key required — runs locally |
| Hinglish/Hindi translation | `SARVAM_API_KEY` | Required for Hinglish mode |
| Summarization, extraction, RAG | `MISTRAL_API_KEY` | Required |

### ⚠️ Security

- Never commit your `.env` file — it's already excluded via `.gitignore`.
- Never put API keys directly inside Python source code.
- If a key is accidentally committed to GitHub, revoke it immediately and generate a new one.

---

## ▶️ Running VideoMind AI

```bash
python -m streamlit run app.py
```

or

```bash
streamlit run app.py
```

The application will normally be available at `http://localhost:8501`.

---

## 🧪 Running the CLI Pipeline

```bash
python main.py
```

You'll be prompted for:

```
Enter YouTube URL or local file path:
Language (english/hinglish):
```

---

## 🧪 Testing

```bash
python test.py
```

The test pipeline processes the configured YouTube URL and generates transcript, title, summary, action items, key decisions, and open questions.

> Update the URL in `test.py` before using it for another video.

---

## 📊 Configuration Reference

### Embedding Pipeline

```
Transcript → RecursiveCharacterTextSplitter → 500-char chunks (50-char overlap)
→ HuggingFace Embeddings → ChromaDB
```

### Mistral Configuration

The application uses `mistral-small-latest` via `ChatMistralAI`:

```python
ChatMistralAI(
    model="mistral-small-latest",
    mistral_api_key=os.getenv("MISTRAL_API_KEY"),
    temperature=0.3,
)
```

Different temperatures are used per task:

| Task | Temperature |
|---|---|
| Summarization | 0.3 |
| Extraction | 0.2 |
| RAG | 0.3 |

### Whisper Configuration

Default model: `small`

```env
WHISPER_MODEL=small
```

- Faster CPU inference: `WHISPER_MODEL=base`
- Higher quality: `WHISPER_MODEL=medium`

> Larger models require significantly more computational resources.

### Sarvam Configuration

Default model: `saaras:v2.5`

```env
SARVAM_STT_MODEL=saaras:v2.5
```

The application sends audio to `https://api.sarvam.ai/speech-to-text-translate`. Audio is split into 25-second pieces to remain below the synchronous API's 30-second limit.

---

## 📁 Generated Files

During processing, the application can generate:

- `downloads/` — downloaded YouTube audio and audio chunks
- `vector_db/` — the RAG vector database

> These generated files should not be committed to Git. The repository `.gitignore` excludes generated media and vector database files.

---

## 🐛 Troubleshooting

**Streamlit command not found**

```bash
python -m streamlit run app.py
```

**Mistral API error**

Check that `MISTRAL_API_KEY` is set and valid in `.env`.

**Sarvam API error**

Check that `SARVAM_API_KEY` is set in `.env`. It's required when using Hinglish mode.

**Whisper model download is slow**

The first Whisper execution downloads the selected model — this can take time depending on model size and internet connection.

**ChromaDB problems**

Delete the generated local database:

```bash
rm -rf vector_db/
```

The application will recreate the vector store when processing a new transcript.

**YouTube download issues**

YouTube extraction can occasionally fail because YouTube changes its delivery mechanisms. If yt-dlp reports:

```
HTTP Error 403: Forbidden
```

or

```
No supported JavaScript runtime could be found
```

update yt-dlp:

```bash
uv pip install -U "yt-dlp[default]"
```

Also make sure FFmpeg is installed. For newer YouTube extraction workflows, a supported JavaScript runtime such as Deno may be required depending on the yt-dlp version and extraction method.

**CPU Whisper warning**

```
FP16 is not supported on CPU; using FP32 instead
```

This is a warning, not a failure — Whisper automatically uses FP32 on CPU and transcription still works.

---

## 🧠 Why RAG?

A meeting transcript can be very long. Instead of sending the entire transcript to the LLM for every question, VideoMind AI stores transcript chunks as vectors. When a user asks a question like *"What deadline did the team agree on?"*, the system searches ChromaDB for the most relevant transcript chunks, and only that relevant context is sent to Mistral.

This provides:

- More relevant answers
- Lower context usage
- Better scalability for long meetings
- Searchable meeting memory
- Context-grounded answers

---

## 🚀 Future Improvements

- 🎤 Real-time meeting transcription
- 👥 Speaker diarization
- 🗣️ Speaker identification
- 🌍 More Indian language support
- 📧 Email action items
- 📅 Calendar integration
- 🔔 Action-item reminders
- 📊 Meeting analytics
- 🧠 Persistent meeting memory
- 🔍 Better semantic search
- 🎥 Video scene understanding
- ☁️ Cloud deployment
- 🔐 User authentication
- 👥 Multi-user workspaces
- 📱 Mobile-friendly interface
- 📄 Improved PDF reports
- ⏱️ Timestamp-aware answers
- 📝 Transcript editing
- 📌 Clickable evidence/citations for RAG answers

### 🔮 Planned Architecture

The long-term goal is to evolve VideoMind AI from a meeting summarizer into a general-purpose AI Video Agent.

```
                    VideoMind AI
                         │
        ┌────────────────┼────────────────┐
        │                │                │
      Video            Audio           YouTube
        │                │                │
        └────────────────┼────────────────┘
                          ▼
                  Multimodal Input
                          ▼
                 Speech / Vision AI
                          ▼
                 Video Understanding
                          ▼
                  Knowledge Layer
                          ▼
              ┌───────────┴───────────┐
              │                       │
         Summarization              RAG
              │                       │
              ▼                       ▼
        Meeting Insights         Video Chat
              │                       │
              └───────────┬───────────┘
                           ▼
                       AI Agent
                           ▼
              Actions / Decisions /
              Questions / Insights
```

---

## 📈 Current Project Status

**Status: 🟢 Working**

Implemented capabilities:

- [x] YouTube URL input
- [x] Local audio/video input
- [x] YouTube audio extraction
- [x] Audio conversion
- [x] Audio chunking
- [x] Local Whisper transcription
- [x] Hinglish transcription/translation using Sarvam AI
- [x] Mistral integration
- [x] LangChain LCEL
- [x] Automatic meeting title generation
- [x] Meeting summarization
- [x] Action-item extraction
- [x] Key-decision extraction
- [x] Open-question extraction
- [x] ChromaDB vector storage
- [x] HuggingFace embeddings
- [x] RAG pipeline
- [x] Meeting Q&A
- [x] Streamlit UI
- [x] Local vector persistence
- [x] CLI pipeline

---

## 🌟 Why VideoMind AI?

Most meeting tools stop at transcription. VideoMind AI goes one step further:

```
Video → Transcription → Understanding → Summary → Decisions
→ Action Items → Knowledge Base → Ask Questions
```

Instead of simply reading a transcript, users can interact with their meeting as a searchable knowledge source.

---

## 👨‍💻 Project

**VideoMind AI**
GitHub: [github.com/chetankumar36/VideoMind-AI](https://github.com/chetankumar36/VideoMind-AI)

---

## 📜 Author

**Chetan Kumar N k**

---

## 🙌 Acknowledgements

This project is built using several open-source and AI technologies:

OpenAI Whisper · LangChain · Mistral AI · Sarvam AI · ChromaDB · HuggingFace · Streamlit · yt-dlp · PyDub · FFmpeg

---

## ⭐ Star the Repository

If you find VideoMind AI useful, consider giving the repository a ⭐ on GitHub.