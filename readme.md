# 🎬 VideoMind AI

### AI-Powered Video & Meeting Intelligence Assistant

VideoMind AI is an AI-powered video and meeting assistant built with Python.

It transforms YouTube videos and uploaded audio/video files into structured, searchable knowledge. The application can extract audio, transcribe speech locally using Whisper, translate Hinglish/Hindi speech into English using Sarvam AI, summarize the content using Mistral, extract action items, key decisions and open questions, and provide an interactive RAG-based chat interface for asking questions about the processed meeting.

The application is built with **Python, Streamlit, Whisper, Sarvam AI, LangChain, Mistral, ChromaDB and HuggingFace embeddings**.

---

## ✨ Features

### 🎥 YouTube Video Processing

Provide a YouTube URL and VideoMind AI will:

1. Download the best available audio using `yt-dlp`
2. Convert the audio to WAV
3. Convert audio to mono 16 kHz when processing uploaded files
4. Split long audio into manageable chunks
5. Send the chunks through the selected transcription pipeline

---

### 📁 Local File Processing

VideoMind AI can also process local audio/video files.

Supported input depends on the FFmpeg/PyDub codecs available on the system.

Typical formats include:

- WAV
- MP3
- MP4
- M4A
- WebM
- Other FFmpeg-supported formats

---

### 🎙️ Local Whisper Transcription

For English transcription, VideoMind AI uses OpenAI Whisper locally.

```text
Audio
  ↓
Whisper
  ↓
English Transcript


The Whisper model can be configured using:

WHISPER_MODEL=small

Available Whisper model choices include:

tiny
base
small
medium
large

Larger models generally provide better transcription quality but require more RAM/CPU/GPU resources.

🌐 Hinglish / Hindi → English

For Hinglish transcription and translation, VideoMind AI uses Sarvam AI's Speech-to-Text Translate API.

The application sends short audio pieces to the API and receives an English transcript.

The current implementation uses:

saaras:v2.5

The audio is split into 25-second pieces before being sent to Sarvam because the synchronous API has a 30-second audio limitation.

Pipeline:

Hinglish / Hindi Audio
        ↓
25-second audio pieces
        ↓
Sarvam AI
        ↓
English Transcript
🧠 AI Meeting Analysis

After transcription, the transcript is processed using Mistral through LangChain.

The application uses:

mistral-small-latest

Mistral is used for:

Meeting title generation
Meeting summarization
Action-item extraction
Key-decision extraction
Open-question extraction
RAG question answering
📋 Automatic Summary

VideoMind AI generates a professional meeting summary from the transcript.

Long transcripts are split into smaller sections before summarization.

Full Transcript
      ↓
Transcript Chunks
      ↓
Individual Summaries
      ↓
Combined Summary
      ↓
Final Meeting Summary

This map-and-combine approach allows the application to handle transcripts that are too large to send to the LLM in one request.

🏷️ Automatic Meeting Title

The application generates a short professional title from the transcript.

For example:

Quarterly Product Planning Meeting

The title generation prompt limits the title to approximately 8 words.

✅ Action Item Extraction

VideoMind AI extracts actionable tasks from the meeting.

For each action item it attempts to identify:

Task description
Owner
Deadline

Example:

1. Prepare the project report
   Owner: Rahul
   Deadline: Friday


2. Contact the client
   Owner: Priya
   Deadline: Not specified
🔑 Key Decision Extraction

The system identifies important decisions made during the meeting.

Example:

1. The project deadline was moved to March 15.
2. The team decided to use PostgreSQL.
3. The next client meeting will be held on Monday.
❓ Open Questions

VideoMind AI also extracts unresolved questions and topics requiring follow-up.

Example:

1. Who will handle the production deployment?
2. What is the final budget?
3. When will the client provide the required assets?
🔎 RAG — Chat With Your Meeting

One of the main features of VideoMind AI is the ability to chat with the processed meeting transcript.

The application uses a Retrieval-Augmented Generation (RAG) pipeline.

RAG Architecture
Transcript
    ↓
Text Splitting
    ↓
Embeddings
    ↓
ChromaDB
    ↓
Similarity Search
    ↓
Relevant Transcript Chunks
    ↓
Mistral
    ↓
Answer
🗄️ Vector Database

VideoMind AI uses ChromaDB as the local vector database.

The transcript is split into chunks using:

RecursiveCharacterTextSplitter

Current configuration:

Chunk size:     500 characters
Chunk overlap:   50 characters

Each chunk is converted into an embedding using:

all-MiniLM-L6-v2

through HuggingFace embeddings.

The vectors are stored locally in:

vector_db/

The vector_db/ directory is intentionally excluded from Git because it is generated application data.

💬 Ask Questions About the Meeting

After processing a video, users can ask questions such as:

What were the main topics discussed?


What did Rahul agree to do?


What were the key decisions?


Who is responsible for the deployment?


What is the project deadline?


What problems were discussed?


What are the next steps?


Was a budget discussed?

The RAG system retrieves relevant transcript sections and sends them to Mistral.

The model is instructed to answer only from the retrieved meeting context.

If the information cannot be found, it responds:

I could not find this information in the meeting transcript.
🔗 LangChain LCEL

VideoMind AI uses LangChain Expression Language (LCEL) to build the LLM and RAG pipelines.

For example, the RAG pipeline conceptually follows:

Question
   ↓
Retriever
   ↓
Relevant Documents
   ↓
Prompt
   ↓
Mistral
   ↓
Answer

The project uses LangChain components including:

ChatPromptTemplate
StrOutputParser
RunnablePassthrough
RunnableLambda
Chroma retrievers
Mistral chat models
🖥️ Streamlit Interface

The user interface is built using Streamlit.

The UI provides a graphical interface for interacting with the AI video assistant.

The application includes:

Video/URL input
Language selection
Processing status
Transcript display
AI-generated title
Meeting summary
Action items
Key decisions
Open questions
RAG chat
Export functionality

The interface uses a custom dark-themed design with animated UI elements and cards.

🏗️ System Architecture
                         VideoMind AI
                              │
             ┌────────────────┴────────────────┐
             │                                 │
       YouTube URL                         Local File
             │                                 │
             ▼                                 ▼
          yt-dlp                           PyDub
             │                                 │
             └──────────────┬──────────────────┘
                            ▼
                       Audio / WAV
                            │
                            ▼
                     Audio Chunking
                            │
                 ┌──────────┴──────────┐
                 │                     │
             English                Hinglish
                 │                     │
                 ▼                     ▼
             Whisper              Sarvam AI
                 │                     │
                 └──────────┬──────────┘
                            ▼
                       Transcript
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
          Summary       Extraction       RAG
             │              │              │
             │        ┌─────┼─────┐        │
             │        │     │     │        │
             │     Actions Decisions Q's   │
             │                              │
             │                              ▼
             │                          Embeddings
             │                              │
             │                           ChromaDB
             │                              │
             │                              ▼
             │                         Similarity Search
             │                              │
             └──────────────┬───────────────┘
                            ▼
                          Mistral
                            │
                            ▼
                       Streamlit UI
                            │
                            ▼
                    User / Meeting Insights
📂 Project Structure
VideoMind-AI/
│
├── app.py
├── main.py
├── test.py
├── Requirements.txt
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
📌 File Responsibilities
app.py

Main Streamlit user interface.

Responsible for:

UI
Input handling
Processing workflow
Displaying transcript and AI results
RAG chat
User interaction
main.py

CLI/application pipeline entry point.

The main pipeline is:

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
utils/audio_processor.py

Responsible for:

YouTube audio downloading
Audio conversion
WAV generation
Audio normalization
Audio chunking

Main functions:

download_youtube_audio()
convert_to_wav()
chunk_audio()
process_input()
core/transcriber.py

Responsible for speech-to-text.

It supports:

English → Whisper
Hinglish → Sarvam AI

Main functions:

load_model()
transcribe_chunk_whisper()
transcribe_chunk_sarvam()
transcribe_chunk()
transcribe_all()
core/summarizer.py

Responsible for:

Transcript splitting
Partial summaries
Final summary generation
Meeting title generation

Uses:

LangChain
Mistral
LCEL
core/extractor.py

Responsible for extracting:

Action Items
Key Decisions
Open Questions
core/vector_store.py

Responsible for:

Creating embeddings
Splitting transcript text
Creating ChromaDB collections
Persisting vectors
Creating retrievers

Embedding model:

all-MiniLM-L6-v2
core/rag_engine.py

Responsible for the RAG pipeline.

Main functions:

build_rag_chain()
load_rag_chain()
ask_question()
🛠️ Technology Stack
Component	Technology
Programming Language	Python
UI	Streamlit
Video Download	yt-dlp
Audio Processing	PyDub
Audio Backend	FFmpeg
Local STT	OpenAI Whisper
Hindi/Hinglish Translation	Sarvam AI
LLM	Mistral
LLM Framework	LangChain
RAG	LangChain LCEL
Vector Database	ChromaDB
Embeddings	HuggingFace Sentence Transformers
Environment Variables	python-dotenv
HTTP Requests	Requests
PDF Support	ReportLab / FPDF2
⚙️ Requirements

Recommended:

Python 3.10+

The project dependencies are listed in:

Requirements.txt
🚀 Installation
1. Clone the repository
git clone https://github.com/chetankumar36/VideoMind-AI.git

Enter the project:

cd VideoMind-AI
2. Create a Virtual Environment
Windows
python -m venv .venv

Activate it:

.venv\Scripts\Activate.ps1

You should see:

(.venv)

or your environment name in the terminal.

📦 Install Dependencies

Using pip:

pip install -r Requirements.txt

Using uv:

uv pip install -r Requirements.txt
🎧 FFmpeg Installation

FFmpeg is required for audio/video processing.

Check whether FFmpeg is installed:

ffmpeg -version

If it is not installed, install FFmpeg and make sure the FFmpeg executable is available in your system PATH.

VideoMind AI uses FFmpeg indirectly through PyDub and ffmpeg-python.

🔐 Environment Variables

Create a file named:

.env

in the project root.

Example:

MISTRAL_API_KEY=your_mistral_api_key


SARVAM_API_KEY=your_sarvam_api_key


WHISPER_MODEL=small


SARVAM_STT_MODEL=saaras:v2.5
Required API Keys

For English/local transcription:

WHISPER_MODEL=small

No transcription API key is required because Whisper runs locally.

For Hinglish/Hindi translation:

SARVAM_API_KEY=your_sarvam_api_key

For summarization, extraction and RAG:

MISTRAL_API_KEY=your_mistral_api_key
⚠️ Security

Never commit your .env file.

Your .gitignore already excludes:

.env

Never put API keys directly inside Python source code.

If an API key is accidentally committed to GitHub, revoke it immediately and generate a new key.

▶️ Running VideoMind AI

Start the Streamlit application:

python -m streamlit run app.py

Or:

streamlit run app.py

The application will normally be available at:

http://localhost:8501
🧪 Running the CLI Pipeline

The project also includes a command-line pipeline.

Run:

python main.py

You will be asked:

Enter YouTube URL or local file path:

Then:

Language (english/hinglish):

For example:

Language: english

or:

Language: hinglish
🧪 Testing

The project contains:

test.py

Run:

python test.py

The test pipeline processes the configured YouTube URL and generates:

Transcript
Title
Summary
Action items
Key decisions
Open questions

Update the URL in test.py before using it for another video.

🔄 Complete Processing Pipeline

When a user submits a video, the application follows this workflow:

1. User provides YouTube URL / Local File
                    ↓
2. Audio extraction
                    ↓
3. WAV conversion
                    ↓
4. Audio chunking
                    ↓
5. Transcription
                    ↓
        ┌───────────┴───────────┐
        ↓                       ↓
     Whisper                Sarvam AI
    English                Hinglish
        │                       │
        └───────────┬───────────┘
                    ↓
              Full Transcript
                    ↓
             Mistral Analysis
                    ↓
       ┌────────────┼────────────┐
       ↓            ↓            ↓
    Summary      Decisions    Actions
                    │
                    ↓
             Open Questions
                    │
                    ↓
             RAG Vector Store
                    │
                    ▼
                ChromaDB
                    │
                    ▼
             User Questions
                    │
                    ▼
                Retriever
                    │
                    ▼
              Relevant Chunks
                    │
                    ▼
                 Mistral
                    │
                    ▼
                 Answer
🧠 Why RAG?

A meeting transcript can be very long.

Instead of sending the entire transcript to the LLM for every question, VideoMind AI stores transcript chunks as vectors.

When a user asks a question:

"What deadline did the team agree on?"

the system searches ChromaDB for the most relevant transcript chunks.

Only the relevant context is sent to Mistral.

This provides:

More relevant answers
Lower context usage
Better scalability for long meetings
Searchable meeting memory
Context-grounded answers
📊 Embedding Pipeline

The current embedding model is:

all-MiniLM-L6-v2

The transcript is processed as:

Transcript
    ↓
RecursiveCharacterTextSplitter
    ↓
500-character chunks
    ↓
50-character overlap
    ↓
HuggingFace Embeddings
    ↓
ChromaDB
🧩 Mistral Configuration

The application currently uses:

mistral-small-latest

The model is accessed through:

ChatMistralAI

Example configuration:

ChatMistralAI(
    model="mistral-small-latest",
    mistral_api_key=os.getenv("MISTRAL_API_KEY"),
    temperature=0.3,
)

Different temperatures are used for different tasks.

For example:

Summarization: 0.3
Extraction: 0.2
RAG: 0.3
🎙️ Whisper Configuration

The default Whisper model is:

small

Change it using:

WHISPER_MODEL=small

For faster CPU inference:

WHISPER_MODEL=base

For higher quality:

WHISPER_MODEL=medium

Larger models require significantly more computational resources.

🌐 Sarvam Configuration

The default Sarvam model is:

saaras:v2.5

Configure it with:

SARVAM_STT_MODEL=saaras:v2.5

The application currently sends audio to:

https://api.sarvam.ai/speech-to-text-translate

Audio is split into 25-second pieces to remain below the synchronous API's 30-second limit.

📁 Generated Files

During processing, the application can generate:

downloades/

for downloaded YouTube audio.

Audio chunks are generated alongside the WAV input.

The RAG database is stored in:

vector_db/

These generated files should not be committed to Git.

The repository .gitignore excludes generated media and vector database files.

⚠️ YouTube Download Issues

YouTube extraction can occasionally fail because YouTube changes its delivery mechanisms.

If yt-dlp reports an error such as:

HTTP Error 403: Forbidden

or:

No supported JavaScript runtime could be found

update yt-dlp:

uv pip install -U "yt-dlp[default]"

Also make sure FFmpeg is installed.

For newer YouTube extraction workflows, a supported JavaScript runtime such as Deno may be required depending on the yt-dlp version and extraction method.

⚠️ CPU Whisper Warning

When Whisper runs on a CPU, you may see:

FP16 is not supported on CPU; using FP32 instead

This is a warning, not a failure.

Whisper automatically uses FP32 on CPU.

For example:

Whisper model loaded.
FP16 is not supported on CPU; using FP32 instead

means transcription is still working.

🐛 Troubleshooting
Streamlit command not found

Use:

python -m streamlit run app.py

instead of:

streamlit run app.py
Mistral API error

Check:

MISTRAL_API_KEY=...

and make sure the key is valid.

Sarvam API error

Check:

SARVAM_API_KEY=...

If using Hinglish mode, the Sarvam API key is required.

Whisper model download

The first Whisper execution downloads the selected model.

This may take some time depending on the model size and internet connection.

ChromaDB problems

Delete the generated local database:

vector_db/

and run the application again.

The application will recreate the vector store when processing a new transcript.

🚀 Future Improvements

Potential future features include:

🎤 Real-time meeting transcription
👥 Speaker diarization
🗣️ Speaker identification
🌍 More Indian language support
📧 Email action items
📅 Calendar integration
🔔 Action-item reminders
📊 Meeting analytics
🧠 Persistent meeting memory
🔍 Better semantic search
🎥 Video scene understanding
☁️ Cloud deployment
🔐 User authentication
👥 Multi-user workspaces
📱 Mobile-friendly interface
📄 Improved PDF reports
⏱️ Timestamp-aware answers
📝 Transcript editing
📌 Clickable evidence/citations for RAG answers
🔮 Planned Architecture

The long-term goal is to evolve VideoMind AI from a meeting summarizer into a general-purpose AI Video Agent.

                    VideoMind AI
                         │
        ┌────────────────┼────────────────┐
        │                │                │
      Video            Audio           YouTube
        │                │                │
        └────────────────┼────────────────┘
                         ↓
                  Multimodal Input
                         ↓
                 Speech / Vision AI
                         ↓
                 Video Understanding
                         ↓
                 Knowledge Layer
                         ↓
              ┌──────────┴──────────┐
              │                     │
          Summarization           RAG
              │                     │
              ▼                     ▼
        Meeting Insights       Video Chat
              │                     │
              └──────────┬──────────┘
                         ↓
                    AI Agent
                         ↓
              Actions / Decisions /
              Questions / Insights
📈 Current Project Status
Status: 🟢 Working

Current implemented capabilities:

 YouTube URL input
 Local audio/video input
 YouTube audio extraction
 Audio conversion
 Audio chunking
 Local Whisper transcription
 Hinglish transcription/translation using Sarvam AI
 Mistral integration
 LangChain LCEL
 Automatic meeting title generation
 Meeting summarization
 Action-item extraction
 Key-decision extraction
 Open-question extraction
 ChromaDB vector storage
 HuggingFace embeddings
 RAG pipeline
 Meeting Q&A
 Streamlit UI
 Local vector persistence
 CLI pipeline
🌟 Why VideoMind AI?

Most meeting tools stop at transcription.

VideoMind AI goes one step further:

Video
  ↓
Transcription
  ↓
Understanding
  ↓
Summary
  ↓
Decisions
  ↓
Action Items
  ↓
Knowledge Base
  ↓
Ask Questions

Instead of simply reading a transcript, users can interact with their meeting as a searchable knowledge source.

👨‍💻 Project

VideoMind AI

GitHub:

https://github.com/chetankumar36/VideoMind-AI

📜 License

Add your preferred license before distributing the project publicly.

For example:

MIT License
🙌 Acknowledgements

This project is built using several open-source and AI technologies:

OpenAI Whisper
LangChain
Mistral AI
Sarvam AI
ChromaDB
HuggingFace
Streamlit
yt-dlp
PyDub
FFmpeg
⭐ Star the Repository

If you find VideoMind AI useful, consider giving the repository a ⭐ on GitHub.

https://github.com/chetankumar36/VideoMind-AI




