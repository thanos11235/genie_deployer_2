# Genie Deployer 2

An intelligent, automated task processing system that receives application briefs, generates complete web applications using LLMs, and deploys them to GitHub Pages—all without manual intervention.

## Overview

Genie Deployer 2 is a FastAPI-based service built for the LLM Code Deployment project. It automates the entire lifecycle of web application development:

- **Receives** task briefs via REST API with verification
- **Generates** production-ready single-page applications using Google's Gemini 2.5 Flash
- **Deploys** to GitHub repositories with automatic GitHub Pages configuration
- **Notifies** evaluation servers with deployment metadata
- **Updates** existing applications with surgical precision in subsequent rounds

## Features

### 🚀 Automated Code Generation
- Uses Gemini 2.5 Flash API to generate complete HTML/CSS/JS applications
- Produces responsive, Tailwind CSS-styled single-page applications
- Generates professional README.md and MIT LICENSE files

### 🔄 Multi-Round Task Handling
- **Round 1**: Full application generation from scratch with repository initialization
- **Round 2+**: Surgical updates that preserve existing functionality while implementing new requirements
- Safe mode prevents destructive rewrites with size-based validation

### 📎 Attachment Support
- Processes image attachments from data URIs or HTTP(S) URLs
- Converts attachments to base64 for Gemini API multimodal prompts
- Saves attachments locally in repository for direct reference in generated HTML

### 🔐 Security & Validation
- Secret-based authentication for all incoming requests
- Nonce verification for evaluation callbacks
- GitHub token authentication for repository operations

### 📊 Comprehensive Logging
- Structured logging to file and console
- `/logs` endpoint for real-time log access
- `/status` endpoint for monitoring active tasks

### ⚡ Concurrent Task Processing
- Configurable semaphore-based concurrency control
- Background task execution with graceful shutdown
- Automatic retry mechanisms for API calls and GitHub operations

## Architecture

```
┌─────────────┐
│   Request   │
│  (JSON)     │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────┐
│   FastAPI Endpoint (/ready)     │
│   - Verify secret                │
│   - Queue background task        │
└──────┬──────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│   LLM Code Generation           │
│   - Parse brief & attachments   │
│   - Call Gemini API              │
│   - Generate HTML/README/LICENSE│
└──────┬──────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│   GitHub Operations             │
│   - Create/clone repository     │
│   - Commit generated files      │
│   - Configure GitHub Pages      │
└──────┬──────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│   Evaluation Notification       │
│   - POST repo metadata          │
│   - Include commit SHA & URL    │
└─────────────────────────────────┘
```

## Installation

### Prerequisites

- Python 3.8+
- Git installed and configured
- GitHub account with personal access token
- Google Gemini API key

### Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/thanos11235/genie_deployer_2.git
   cd genie_deployer_2
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Configure environment variables**

   Create a `.env` file in the project root:
   ```env
   GEMINI_API_KEY=your_gemini_api_key_here
   GITHUB_TOKEN=your_github_personal_access_token
   GITHUB_USERNAME=your_github_username
   STUDENT_SECRET=your_secret_key_for_authentication
   
   # Optional configurations
   LOG_FILE_PATH=logs/app.log
   MAX_CONCURRENT_TASKS=2
   KEEP_ALIVE_INTERVAL_SECONDS=30
   ```

4. **Run the application**
   ```bash
   uvicorn main:app --host 0.0.0.0 --port 8000
   ```

## Usage

### Submitting a Task

Send a POST request to `/ready` with the following JSON structure:

```json
{
  "email": "student@example.com",
  "secret": "your_student_secret",
  "task": "unique-task-id",
  "round": 1,
  "nonce": "unique-nonce-value",
  "brief": "Create a responsive calculator app with Tailwind CSS",
  "evaluation_url": "https://evaluation-server.com/notify",
  "attachments": [
    {
      "name": "logo.png",
      "url": "data:image/png;base64,iVBORw0KGgo..."
    }
  ]
}
```

### Response

The service immediately returns a 200 OK response and processes the task in the background:

```json
{
  "status": "ready",
  "message": "Task unique-task-id received and processing started."
}
```

### Monitoring

Check task status and logs:

```bash
# Get current status
curl http://localhost:8000/status

# View recent logs (last 200 lines)
curl http://localhost:8000/logs?lines=200

# Health check
curl http://localhost:8000/health
```

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/ready` | POST | Submit a new task for processing |
| `/` | GET | Service info and status |
| `/status` | GET | Last received task and active background tasks |
| `/health` | GET | Service health check |
| `/logs` | GET | Retrieve application logs (query param: `lines`) |

## How It Works

### Round 1: Initial Generation

1. **Request Verification**: Validates secret and parses task data
2. **Repository Setup**: Creates new GitHub repository with initial commit
3. **LLM Generation**: Sends brief and attachments to Gemini API
4. **Code Generation**: Produces `index.html`, `README.md`, and `LICENSE`
5. **Deployment**: Commits files, pushes to GitHub, configures Pages
6. **Notification**: POSTs deployment metadata to evaluation URL

### Round 2+: Surgical Updates

1. **Repository Clone**: Fetches existing repository
2. **Context Loading**: Reads current `index.html` for context
3. **Targeted Generation**: LLM performs minimal changes based on brief
4. **Safety Checks**: Validates output isn't destructively different
5. **Deployment**: Commits updates and redeploys to Pages
6. **Notification**: Confirms update completion to evaluation server

### Safety Features

- **Size Validation**: Rejects LLM outputs that are suspiciously smaller than originals
- **Fallback Mechanisms**: Preserves existing files if LLM generation fails
- **Retry Logic**: Exponential backoff for API calls and GitHub operations
- **Content Preservation**: Maintains README/LICENSE unless explicitly updated

## Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `GEMINI_API_KEY` | Yes | - | Google Gemini API key |
| `GITHUB_TOKEN` | Yes | - | GitHub personal access token with repo permissions |
| `GITHUB_USERNAME` | Yes | - | GitHub username for repository creation |
| `STUDENT_SECRET` | Yes | - | Secret key for request authentication |
| `LOG_FILE_PATH` | No | `logs/app.log` | Path to log file |
| `MAX_CONCURRENT_TASKS` | No | `2` | Maximum concurrent task processing |
| `KEEP_ALIVE_INTERVAL_SECONDS` | No | `30` | Heartbeat interval for monitoring |

### GitHub Token Permissions

Your GitHub personal access token needs:
- `repo` (full control of private repositories)
- `workflow` (if using GitHub Actions)

## Project Structure

```
genie_deployer_2/
├── main.py                 # Main application entry point
├── .env                    # Environment configuration (not in repo)
├── requirements.txt        # Python dependencies
├── logs/                   # Application logs
│   └── app.log
└── generated_tasks/        # Local task storage (temporary)
    └── {task-id}/
        ├── index.html
        ├── README.md
        ├── LICENSE
        └── {attachments}
```

## Dependencies

Core dependencies (see `requirements.txt` for versions):

- **FastAPI**: Web framework for API endpoints
- **Pydantic**: Data validation and settings management
- **httpx**: Async HTTP client for API calls
- **GitPython**: Python Git interface
- **uvicorn**: ASGI server

## Error Handling

The service implements comprehensive error handling:

- **API Failures**: Automatic retries with exponential backoff
- **Git Errors**: Detailed logging with cleanup on failure
- **LLM Errors**: Fallback to existing content when generation fails
- **Network Issues**: Retry logic for evaluation server notifications

## Logging

Logs include:
- Request reception and validation
- LLM API calls and responses
- Git operations (clone, commit, push)
- GitHub Pages configuration
- Evaluation server notifications
- Error traces and warnings

Access logs via:
- Console output (stdout)
- Log file at configured path
- `/logs` HTTP endpoint

## Troubleshooting

### Common Issues

**GitHub Pages not deploying**
- Verify repository is public
- Check GitHub token has sufficient permissions
- Allow 1-2 minutes for Pages to build after push

**LLM generation failures**
- Verify Gemini API key is valid and has quota
- Check brief complexity (very long briefs may timeout)
- Review logs for specific API error messages

**Secret mismatch errors**
- Ensure `STUDENT_SECRET` in `.env` matches submission
- Check for whitespace or encoding issues in secret

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing

This project is part of an academic assignment. External contributions are not currently accepted.

## Acknowledgments

Built for the LLM Code Deployment project at the International Institute of Information Technology, Hyderabad (IIIT-H).

---

**Created by**: [thanos11235](https://github.com/thanos11235)  
**Status**: Active Development  
**Last Updated**: October 2025
