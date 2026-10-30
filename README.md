# MyChatGPT
# MyChatGPT

MyChatGPT is a FastAPI chat application with a Gemini-powered assistant, web
search, document uploads and search, and conversation memory.

## Deployments

You can run this project as two independent deployments: one on Render and
another on AWS. Each platform runs its own app instance with its own URL,
environment variables, and local data. The deployments do not share
conversations, uploaded documents, or other application data.

The app currently stores its SQLite databases, uploaded files, and Chroma
vector index in `data/`, `uploads/`, and `chroma_db/` relative to its working
directory. Run one application worker per instance; this app uses local SQLite
and in-process state.

### Required environment variables

Set these variables separately for each deployment:

| Variable | Required | Description |
| --- | --- | --- |
| `GOOGLE_API_KEY` | Yes | API key used by Gemini chat and embeddings. |
| `TAVILY_API_KEY` | For web search | API key used by the Tavily web-search tool. |
| `GEMINI_MODEL` | No | Default Gemini model; defaults to `gemini-3.8-flash`. |

Add secrets through the platform's environment-variable settings. Do not commit
API keys or `.env` files to the repository.

### Deploy to Render

1. Push this repository to GitHub.
2. In Render, create a **New > Web Service** and connect the repository.
3. Select the Python runtime and configure:
   - **Build command:** `pip install -r requirements.txt`
   - **Start command:** `uvicorn app:app --host 0.0.0.0 --port $PORT`
4. Add the required environment variables above in the service's **Environment**
   settings.
5. Deploy the service and open its Render URL.

Render's default service filesystem is ephemeral. Files in `data/`, `uploads/`,
and `chroma_db/` can be lost when the service restarts or is redeployed. If
those files must persist, configure persistent storage and update the
application to store all three directories on the mounted disk before relying
on the deployment for durable data.

### Deploy to AWS (EC2)

This deployment uses a separate EC2 instance and its own local data.

1. Launch an EC2 instance with a supported Python version and attach durable
   storage (for example, an EBS volume) for the application and its data.
2. Configure the instance's security group to allow SSH from your trusted IP.
   For public access, put the app behind a reverse proxy such as Nginx and allow
   HTTPS traffic; avoid exposing the development server directly to the
   internet.
3. Clone this repository onto the instance, then install dependencies:

   ```bash
   git clone <your-repository-url>
   cd MyChatGPT
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```

4. Set `GOOGLE_API_KEY` and, if using web search, `TAVILY_API_KEY` in the
   instance's service environment or another secure secrets store. Optionally
   set `GEMINI_MODEL`.
5. Start the app from the repository directory with one worker:

   ```bash
   .venv/bin/uvicorn app:app --host 127.0.0.1 --port 8080
   ```

   Configure a process manager such as systemd to start and restart the app
   automatically, and configure Nginx to proxy HTTPS requests to
   `127.0.0.1:8080`.
6. Open the public HTTPS URL configured for the instance.

Keep the app's working directory and its `data/`, `uploads/`, and `chroma_db/`
directories on durable storage. Back them up regularly; the SQLite databases
and uploaded documents are not automatically copied to Render or another AWS
instance.

## Run locally

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Set `GOOGLE_API_KEY` (and optionally `TAVILY_API_KEY` and `GEMINI_MODEL`) in
your local environment or an uncommitted `.env` file, then run:

```bash
uvicorn app:app --reload
```

Open `http://127.0.0.1:8000` in a browser.
