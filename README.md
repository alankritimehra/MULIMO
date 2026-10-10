# MULIMO — Multimodal Content Moderation

MULIMO is a multimodal content moderation application built with Python, FastAPI, Gradio, and Google Gemini. It analyzes text, images, audio, and video to identify potentially unsafe or inappropriate content before it reaches an AI-powered customer service agent.

The project combines structured AI outputs, API-based moderation, and observability with OpenTelemetry and Arize Phoenix.

## Features

- **Text moderation:** Detects personally identifiable information (PII), unfriendly language, and unprofessional communication.
- **Image moderation:** Checks for PII, disturbing content, and low-quality images.
- **Audio moderation:** Transcribes audio and evaluates it for PII, unfriendly language, and unprofessional communication.
- **Video moderation:** Evaluates video content for PII, disturbing content, and low-quality footage.
- **AI-powered customer simulation:** Uses Google Gemini through Pydantic AI to simulate an ACME customer in a customer service training scenario.
- **API authentication:** Protects API endpoints using bearer-token authentication.
- **Interactive interface:** Provides a Gradio chat interface for text and media uploads.
- **Observability:** Uses OpenTelemetry and Arize Phoenix to trace moderation operations and AI interactions.
- **Structured outputs:** Uses Pydantic models to represent moderation results consistently.

## Tech Stack

| Technology | Purpose |
|---|---|
| Python 3.12+ | Core application language |
| FastAPI | REST API for moderation |
| Gradio | Interactive chat interface |
| Google Gemini | AI model for moderation and customer simulation |
| Pydantic AI | AI agent orchestration and structured outputs |
| Pydantic | Data validation and result schemas |
| OpenTelemetry | Application tracing |
| Arize Phoenix | Trace visualization and observability |
| Pytest | Testing |

## Architecture

MULIMO uses a layered architecture in which uploaded content is moderated before being passed to the simulated customer agent.

```text
                   User
                    |
                    v
             Gradio Chat UI
                    |
                    v
           FastAPI Moderation API
                    |
          +---------+---------+
          |         |         |
          v         v         v
         Text     Image     Audio / Video
          |         |         |
          +---------+---------+
                    |
                    v
           Structured Moderation
                 Results
                    |
             Content Decision
                    |
          +---------+---------+
          |                   |
          v                   v
      Flagged Content     Safe Content
          |                   |
          v                   v
       Blocked         Customer AI Agent
                              |
                              v
                        Gemini Response

       OpenTelemetry + Arize Phoenix
          Trace application activity
```

The moderation API evaluates incoming content and returns structured results. The Gradio application uses these results to decide whether the content can proceed to the customer simulation.

## Project Structure

```text
MULIMO/
├── multimodal_moderation/
│   ├── agents/
│   │   ├── text_agent.py
│   │   ├── image_agent.py
│   │   ├── audio_agent.py
│   │   └── video_agent.py
│   ├── types/
│   │   ├── moderation_result.py
│   │   └── model_choice.py
│   ├── app.py
│   ├── fastapi_app.py
│   ├── gradio_app.py
│   ├── customer_agent.py
│   ├── tracing.py
│   ├── env.py
│   └── utils.py
├── tests/
├── pyproject.toml
├── .env.example
└── README.md
```

*Note: This structure represents the main files discussed in the project. Your actual repository may contain additional files or directories.*

## Getting Started

### Prerequisites

Before running the project, ensure you have:

- Python 3.12 or later
- A Google Gemini API key compatible with your configured model endpoint
- Access to the configured Phoenix instance if you want to view traces
- Git and a terminal

### 1. Clone the repository

```bash
git clone https://github.com/alankritimehra/MULIMO.git
cd MULIMO
```

### 2. Create a virtual environment

**Windows:**

```bash
python -m venv .venv
.venv\Scripts\activate
```

**macOS/Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

Install the project and its dependencies using pip:

```bash
python -m pip install --upgrade pip
pip install -e .
```

If you use the project's configured development tool, you can instead install it with the appropriate command for that tool.

### 4. Configure environment variables

Create a `.env` file in the project root. Configure the required values using your own credentials.

```env
USER_API_KEY=your_user_api_key
GEMINI_API_KEY=your_gemini_api_key
DEFAULT_GOOGLE_MODEL=gemini-2.5-flash-lite
EVAL_JUDGE_MODEL=gemini-2.5-flash-lite
EVAL_NUM_REPEATS=5
API_BASE_URL=http://localhost:8000
PHOENIX_URL=http://127.0.0.1:6006
```

The environment configuration must contain non-empty values for `USER_API_KEY`, `GEMINI_API_KEY`, and `DEFAULT_GOOGLE_MODEL`.

**Security:** Never commit your real `.env` file or API keys to GitHub. Keep `.env` in `.gitignore` and provide placeholders only in `.env.example`.

### 5. Run the application

The project defines the following command-line entry points.

**Option A: Start the complete application**

```bash
multimodal-moderation
```

This starts the Phoenix interface and launches the moderation API and Gradio application.

**Option B: Start the services separately**

Terminal 1 — Moderation API:

```bash
multimodal-moderation-api
```

Terminal 2 — Gradio interface:

```bash
multimodal-moderation-chat
```

If you start the services separately, ensure that Phoenix is available at the configured URL before expecting traces to appear.

### 6. Open the interfaces

When running locally, the default addresses are:

- **Gradio chat:** http://localhost:7860
- **FastAPI API:** http://localhost:8000
- **FastAPI documentation:** http://localhost:8000/docs
- **Phoenix UI:** http://localhost:6006

The API endpoints require bearer-token authentication, including the health endpoint in the current implementation.

## API Endpoints

All endpoints use bearer-token authentication through the configured `USER_API_KEY`.

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/v1/moderate_text` | Moderates text content |
| POST | `/api/v1/moderate_image_file` | Moderates an image upload |
| POST | `/api/v1/moderate_audio_file` | Moderates an audio upload |
| POST | `/api/v1/moderate_video_file` | Moderates a video upload |
| GET | `/api/v1/health` | Returns the API health status |

### Example: Moderate text

```bash
curl -X POST "http://localhost:8000/api/v1/moderate_text" \
  -H "Authorization: Bearer your_user_api_key" \
  -H "Content-Type: application/json" \
  -d "{\"text\":\"Your text to moderate goes here.\"}"
```

The response is a structured moderation result containing applicable flags and a rationale.

## Moderation Criteria

| Modality | Checks |
|---|---|
| Text | PII, unfriendly language, unprofessional language |
| Image | PII, disturbing content, low image quality |
| Audio | Transcription, PII, unfriendly language, unprofessional language |
| Video | PII, disturbing content, low video quality |

The moderation agents return structured outputs using Pydantic models. The application uses these outputs to determine whether incoming content should be blocked.

## Observability with Arize Phoenix

MULIMO uses OpenTelemetry and OpenInference instrumentation to capture tracing information.

Tracing helps inspect operations such as:

- Moderation requests
- Individual chat turns
- AI model calls
- Media metadata associated with moderation spans

The traces are exported to the configured Phoenix endpoint, allowing developers to investigate application behavior and troubleshoot issues.

## Testing

The repository includes tests for moderation result schemas, environment setup, Gradio behavior, and the text, image, audio, and video agents.

Run the test suite with:

```bash
pytest
```

Some tests may require environment variables, model credentials, or access to external services, depending on their implementation.

## Security Considerations

- Store API keys in environment variables, not source code.
- Use HTTPS and stronger credential management when deploying beyond local development.
- Apply appropriate upload-size limits and file validation.
- Avoid retaining sensitive uploaded media or tracing data longer than necessary.
- Review logs and traces to ensure they do not expose confidential user content.
- Add appropriate request timeouts and error handling for external services.

## Future Improvements

Potential improvements include:

- Adding confidence scores to structured moderation results.
- Improving handling of uncertain moderation decisions.
- Adding file-size limits and stronger media validation at the API layer.
- Improving tracing error reporting and uploaded-media cleanup.
- Expanding automated tests for edge cases and model failures.
- Adding deployment configuration and production monitoring.

## License

Add the license applicable to this repository before distributing or reusing the project.
