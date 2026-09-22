# Comprehensive Operational Instructions & Execution Guide

## Multi-Agent Systems with Google Agent Development Kit (ADK)

---

## 📋 Table of Contents
1. [Prerequisites & GCP Configuration](#1-prerequisites--gcp-configuration)
2. [Local Environment Setup](#2-local-environment-setup)
3. [Configuration & Environment Variables](#3-configuration--environment-variables)
4. [How to Run the Code: Execution Modes](#4-how-to-run-the-code-execution-modes)
   - [Option A: Interactive Web UI (`adk web`)](#option-a-interactive-web-ui-adk-web)
   - [Option B: Interactive Terminal CLI (`adk run`)](#option-b-interactive-terminal-cli-adk-run)
5. [Why We Choose Which Option (Execution Trade-Offs)](#5-why-we-choose-which-option-execution-trade-offs)
6. [Session Persistence & Storage Options](#6-session-persistence--storage-options)
7. [Step-by-Step Testing & Verification Walkthroughs](#7-step-by-step-testing--verification-walkthroughs)
   - [Walkthrough 1: Parent & Subagents (Travel Assistant)](#walkthrough-1-parent--subagents-travel-assistant)
   - [Walkthrough 2: Workflow Agents (Film Pitch Pipeline)](#walkthrough-2-workflow-agents-film-pitch-pipeline)
8. [Troubleshooting & Common Pitfalls](#8-troubleshooting--common-pitfalls)

---

## 1. Prerequisites & GCP Configuration

Before running this project, ensure you have the following requirements configured:

### 1.1 Python Version
- **Python 3.10, 3.11, or 3.12** is required.
- Check your local version:
  ```bash
  python3 --version
  ```

### 1.2 Google Cloud Platform (GCP) Project
1. A valid GCP account with an active billing project.
2. Enable required Cloud APIs:
   ```bash
   gcloud services enable \
       aiplatform.googleapis.com \
       logging.googleapis.com
   ```
3. Set your active GCP project:
   ```bash
   gcloud config set project YOUR_PROJECT_ID
   ```
4. Authenticate your local environment using **Application Default Credentials (ADC)**:
   ```bash
   gcloud auth application-default login
   ```
   *Note: This generates credentials stored at `~/.config/gcloud/application_default_credentials.json`, which the Google ADK and Vertex AI SDK read automatically.*

---

## 2. Local Environment Setup

### 2.1 Clone the Repository
```bash
git clone https://github.com/sayem041088/Multi-Agent-System-with-ADK.git
cd Multi-Agent-System-with-ADK
```

### 2.2 Create and Activate a Virtual Environment
Isolating dependencies in a dedicated virtual environment prevents package conflicts:
```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 2.3 Install Production Dependencies
Install all required libraries specified in [requirements.txt](requirements.txt):
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

Installed packages include:
- `google-adk==1.27.4`: Core agent orchestration, state management, and CLI runtime.
- `langchain-community==0.3.27`: Community tool integrations.
- `wikipedia==1.4.0`: Real-time encyclopedia lookup engine.
- `google-cloud-logging`: Enterprise log streaming and observability.
- `python-dotenv`: Environment variable loader.

---

## 3. Configuration & Environment Variables

Copy the provided `.env.example` template into a working `.env` file:
```bash
cp .env.example .env
```

Open `.env` and verify the values:
```ini
# Enables Vertex AI integration for the Google GenAI SDK
GOOGLE_GENAI_USE_VERTEXAI=TRUE

# Your Google Cloud Project ID with billing enabled
GOOGLE_CLOUD_PROJECT="your-project-id"

# Vertex AI Model Location (global, us-central1, etc.)
GOOGLE_CLOUD_LOCATION=global

# Model selection for all agents
MODEL="gemini-3.8-flash"
```

### Explanation of Environment Variables:
- `GOOGLE_GENAI_USE_VERTEXAI=TRUE`: Tells the ADK to route calls through Google Cloud Vertex AI infrastructure rather than standard consumer AI Studio API keys.
- `GOOGLE_CLOUD_PROJECT`: Maps resource usage and quota to your cloud billing account.
- `MODEL="gemini-3.8-flash"`: Specifies the Gemini 3.8 Flash model, offering low latency, large context handling, and high tool-calling reliability.

---

## 4. How to Run the Code: Execution Modes

Google ADK provides two primary operational modes: **Web UI** and **Terminal CLI**.

---

### Option A: Interactive Web UI (`adk web`)

The Web UI starts an in-memory FastAPI server serving a graphical multi-agent workstation.

#### Command:
```bash
adk web .
```

To run on a custom port or specify a custom host:
```bash
adk web --host 0.0.0.0 --port 8080 .
```

#### Access:
1. Open your browser and navigate to **`http://127.0.0.1:8000`** (or your specified port).
2. Use the top navigation bar to select the agent module to test:
   - **`parent_and_subagents`**
   - **`workflow_agents`**
3. Type messages in the chat input and watch the real-time interaction.

#### Features Available in the Web UI:
- **Topology Inspector**: Displays the agent hierarchy, subagent delegations, and parent links.
- **Trace & Tool Execution Viewer**: Expandable cards showing tool inputs, function call arguments, and tool outputs (such as Wikipedia search results or state mutations).
- **Session State Inspector**: Live JSON tree showing current variables in `tool_context.state`.

---

### Option B: Interactive Terminal CLI (`adk run`)

The CLI runs a dedicated terminal session directly connected to a specific agent module.

#### Command 1: Run Parent-Subagent Travel Assistant
```bash
adk run parent_and_subagents
```

#### Command 2: Run Film Concept Workflow Pipeline
```bash
adk run workflow_agents
```

#### Useful CLI Flags:
```bash
# Enable verbose debug logs to trace internal agent decisions
adk run --verbose workflow_agents

# Save session history to a JSON file upon exit
adk run --save_session --session_id session_01 workflow_agents

# Resume a previously saved session
adk run --resume session_01.json workflow_agents

# Replay an automated query script non-interactively
adk run --replay test_queries.json workflow_agents
```

---

## 5. Why We Choose Which Option (Execution Trade-Offs)

When deciding between **`adk web`** and **`adk run`**, select based on your operational goal:

| Operational Goal | Recommended Mode | Why We Chose This Option |
| :--- | :--- | :--- |
| **Architectural Auditing & Debugging** | **`adk web`** | The Web UI visually exposes intermediate agent handoffs, internal tool parameters, and the evolving state dictionary. Debugging loop transitions without visual cues can be difficult in a terminal. |
| **Stakeholder Demonstrations** | **`adk web`** | Provides a clean, modern graphical interface where non-technical stakeholders can observe agent collaboration in real time. |
| **Automated Testing & CI/CD Pipelines** | **`adk run`** | The CLI supports headless execution, file replays (`--replay`), and stdout piping, making it suitable for automated regression testing. |
| **Rapid Prototyping & Developer Iteration** | **`adk run`** | Starts instantly in the developer's terminal without opening browser windows or binding local network ports. |
| **Performance Profiling** | **`adk run --verbose`** | Emits raw timestamps and stdout logs without browser rendering overhead, providing accurate latency benchmarks. |

---

## 6. Session Persistence & Storage Options

Google ADK allows configuring how session history and state dictionaries are stored between invocations.

### 1. In-Memory Storage (`memory://`)
- **Command**: `adk run --session_service_uri memory:// workflow_agents`
- **Why Choose This**: Zero disk footprint. Best for ephemeral unit tests, automated CI test suites, and temporary scratch sessions.

### 2. Local File/Directory Storage (`.adk/` Default)
- **Command**: `adk run workflow_agents` (uses `--use_local_storage` by default)
- **Why Choose This**: Automatically writes session checkpoints into the local `.adk/` folder. Allows pausing a session and resuming it later.

### 3. SQLite Relational Storage
- **Command**: `adk web --session_service_uri sqlite:///adk_sessions.db .`
- **Why Choose This**: Stores multi-agent sessions in an ACID-compliant local database. Ideal for local multi-user testing and persistent session analysis.

### 4. Vertex AI Agent Engine (`agentengine://`)
- **Command**: `adk run --session_service_uri agentengine://projects/PROJECT/locations/LOCATION/reasoningEngines/ID workflow_agents`
- **Why Choose This**: Enterprise production deployments on Google Cloud. Manages session lifecycles, autoscaling, and cross-region replication automatically.

---

## 7. Step-by-Step Testing & Verification Walkthroughs

### Walkthrough 1: Parent & Subagents (Travel Assistant)

1. Launch the agent:
   ```bash
   adk run parent_and_subagents
   ```
2. **Step 1: Test Exploratory Brainstorming**
   - User Input: `"I want to take a vacation, but I don't know where to go."`
   - Expected Behavior: The `steering` root agent identifies uncertainty and hands off to `travel_brainstormer`, which asks about your interests (e.g., adventure, art, relaxation).
3. **Step 2: Test Direct Planning & Tool Invocation**
   - User Input: `"I want to visit France. Can you suggest some attractions?"`
   - Expected Behavior: The `steering` agent routes to `attractions_planner`, which lists top sites (Eiffel Tower, Louvre Museum).
4. **Step 3: Test State Accumulation**
   - User Input: `"Add the Louvre and Mont Saint-Michel to my list, and show me what I have saved so far."`
   - Expected Behavior: The agent invokes `save_attractions_to_state`, updates `tool_context.state["attractions"]`, and outputs the formatted list from state.

---

### Walkthrough 2: Workflow Agents (Film Pitch Pipeline)

1. Launch the workflow:
   ```bash
   adk run workflow_agents
   ```
2. **Step 1: Prompt Submission**
   - Agent Greeting: `"I will help you write a pitch for a hit movie. What historical figure would you like to create a movie about?"`
   - User Input: `"Ada Lovelace"`
3. **Step 2: Autonomous Execution Stages**
   - **Writers' Room (`LoopAgent`)**:
     - `researcher` queries Wikipedia for Ada Lovelace, Charles Babbage, and the Analytical Engine.
     - `screenwriter` generates the 3-act outline (*"Poetical Science"*).
     - `critic` audits the outline against the 4 cinematic criteria. If improvements are identified, feedback is appended to state and the loop repeats; once satisfied, `critic` calls `exit_loop`.
   - **Preproduction Team (`ParallelAgent`)**:
     - `box_office_researcher` computes theatrical revenue benchmarks ($95M–$135M base case) based on comparable films like *The Imitation Game* and *Hidden Figures*.
     - `casting_agent` recommends actors (Saoirse Ronan as Ada, Jared Harris as Babbage).
   - **File Writer (`file_writer`)**:
     - Calls `write_file` to write `movie_pitches/Poetical Science.txt`.
4. **Step 3: Verify Output Artifact**
   - Confirm the pitch file was written to disk:
     ```bash
     ls -la movie_pitches/
     cat "movie_pitches/Poetical Science.txt"
     ```

---

## 8. Troubleshooting & Common Pitfalls

### 8.1 `DefaultCredentialsError` / Google Auth Failure
- **Symptom**: `google.auth.exceptions.DefaultCredentialsError: Could not automatically determine credentials.`
- **Remedy**: Run `gcloud auth application-default login` to refresh your local token file. Verify that `GOOGLE_CLOUD_PROJECT` in your `.env` matches your active GCP project.

### 8.2 Vertex AI 403 Forbidden / API Not Enabled
- **Symptom**: `google.api_core.exceptions.PermissionDenied: 403 Vertex AI API has not been used in project...`
- **Remedy**: Enable the API via `gcloud services enable aiplatform.googleapis.com` and ensure your IAM role includes `roles/aiplatform.user`.

### 8.3 LoopAgent Reaches `max_iterations=5`
- **Symptom**: The writer loop runs 5 times without early exit.
- **Cause**: The `critic` agent was overly rigorous or the model did not emit the `exit_loop` tool call.
- **Remedy**: Check `callback_logging.py` logs to see critic feedback. `max_iterations=5` ensures the system gracefully advances to preproduction regardless, preventing infinite loops.

### 8.4 Web Server Port 8000 Conflict
- **Symptom**: `OSError: [Errno 98] Address already in use`
- **Remedy**: Specify an alternative port using the `--port` flag:
  ```bash
  adk web --port 8085 .
  ```
