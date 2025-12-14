# QueryBridge

QueryBridge is a low-code tool for building chatbots that answer questions from your own data. Connect a database, upload documents, or point it at a website, then choose an LLM, and you get a working conversational assistant with a chat UI you can embed in any web page.

## Features

- **Chat with your databases.** QueryBridge turns plain-English questions into SQL, runs them, and explains the results.
- **Chat with your documents.** Load text files, Word documents, CSVs, or website content into a vector database and ask questions about them.
- **Bring your own LLM.** Works with OpenAI, Together AI, Ollama (local models), and AI71.
- **Capabilities.** Define custom tasks the bot can perform, such as booking a meeting or filling in a form. The bot collects the required details from the user, then runs the task.
- **Actions.** The bot can read data from and write data to a connected database.
- **Intent detection.** The bot works out what the user wants and picks the right capability automatically.
- **Embeddable chat widget.** Add the chatbot to any website with a short JavaScript snippet.
- **Authentication.** Optional login through Zitadel, with Google sign-in support.
- **Multiple bots.** Create separate bots for different teams or use cases, each with its own data and settings.

## Supported integrations

| Type          | Options                                                           |
| ------------- | ----------------------------------------------------------------- |
| LLM providers | OpenAI, Together AI, Ollama, AI71                                 |
| Databases     | MySQL, PostgreSQL, MariaDB, MSSQL, SQLite, BigQuery, Airtable     |
| Documents     | Document loader (text and Word files), CSV loader, website loader |
| Vector stores | ChromaDB (default), MongoDB                                       |

## How it works

```text
            ┌───────────────────────────────┐
 User ────► │  Chat UI / embeddable widget  │
            └──────────────┬────────────────┘
                           ▼
            ┌───────────────────────────────┐
            │          FastAPI backend      │
            │  intent detection → routing   │
            └───────┬───────────────┬───────┘
                    │               │
        ┌───────────▼───┐   ┌───────▼────────────┐
        │  SQL plugins  │   │  Document plugins  │
        │ (generate and │   │  (vector search in │
        │  run queries) │   │     ChromaDB)      │
        └───────┬───────┘   └───────┬────────────┘
                └───────┬───────────┘
                        ▼
                 LLM writes the answer
```

1. The user asks a question in the chat.
2. QueryBridge detects the intent and decides which data source or capability to use.
3. For databases, it generates and runs a SQL query. For documents, it searches the vector store.
4. The LLM writes the final answer from the results.

## Tech stack

| Layer           | Technology                         |
| --------------- | ---------------------------------- |
| Backend         | Python 3.11, FastAPI, LangChain    |
| Frontend        | React 18, Vite                     |
| Vector store    | ChromaDB (ONNX default embeddings) |
| Database access | SQLAlchemy and native drivers      |
| Auth            | Zitadel                            |
| Deployment      | Docker, Docker Compose, Nginx      |

## Getting started

### Requirements

- Python 3.11+
- Node.js 18+ and npm
- Docker and Docker Compose (only if you want login)
- An API key for your LLM provider, or Ollama running locally

### 1. Clone and install the backend

```bash
git clone <your-repo-url>
cd querybridge
pip install -r requirements.txt
```

Or use Poetry:

```bash
poetry install
```

### 2. Configure

Settings live in `config.yaml`. The default vector store setup is:

```yaml
vector_db:
  name: "chroma"
  params:
    path: "./vector_db"
    embeddings:
      provider: "chroma_default"
```

Copy `.env.example` to `.env` and fill in your values.

### 3. Run the backend

```bash
python main.py --config ./config.yaml llm
```

### 4. Run the frontend

```bash
cd ui
npm install
```

Create `ui/.env` with the backend URL:

```env
VITE_BACKEND_URL=http://localhost:8001
```

Then start the dev server:

```bash
npm run dev
```

**To serve the UI from the backend instead**, set `VITE_BACKEND_URL=""` in `ui/.env`, build the UI, and restart the backend:

```bash
npm run build
cd ..
python main.py --config ./config.yaml llm
```

### 5. Run everything with Docker (optional)

```bash
docker-compose up -d
```

## Optional: login with Zitadel

1. Start Zitadel:

   ```bash
   docker-compose -f zitadel-docker-compose.yaml up -d
   ```

2. Open `localhost:8080` and sign in with the default admin account:
   - Username: `zitadel-admin@zitadel.localhost`
   - Password: `Password1!` (change it after your first login)

3. Create a **service user**:
   - Go to **Users → Service Users → New**.
   - Set **Access Token Type** to **JWT**.
   - Under **Keys**, click **New → Add → Download** to save the key file.

4. In the **Organization** tab, click **Add a Manager**, select the service user, and give it the **Org Owner** role.

5. To enable Google sign-in, add Google as an identity provider in Zitadel, with redirect URI `localhost:8080/idps/callback`.

6. Add these values to `.env`:

   ```env
   CLIENT_PRIVATE_KEY_FILE_PATH="./path/to/downloaded/key.json"
   ZITADEL_TOKEN_URL="http://localhost:8080/oauth/v2/token"
   ZITADEL_DOMAIN="http://localhost:8080"
   ```

## Project structure

```text
├── app/
│   ├── api/            # REST endpoints
│   ├── chain/          # Query and response pipelines
│   ├── loaders/        # LLM providers (OpenAI, Together AI, Ollama, AI71)
│   ├── plugins/        # Data sources (MySQL, PostgreSQL, BigQuery, CSV, website, ...)
│   ├── vectordb/       # ChromaDB and MongoDB vector stores
│   ├── providers/      # Config, caching, reranking, auth
│   ├── services/       # Business logic
│   └── repository/     # Database access
├── ui/                 # React frontend and embeddable chat widget
├── tests/              # Unit and integration tests
├── config.yaml         # Main configuration
├── docker-compose.yml  # Full stack
└── main.py             # Entry point
```

## Troubleshooting

**"Your system has an unsupported version of sqlite3. Chroma requires sqlite3 >= 3.35.0"**

Your system's SQLite is too old for ChromaDB. Either upgrade SQLite, or install `pysqlite3-binary` and add this to the top of `main.py`:

```python
__import__("pysqlite3")
import sys
sys.modules["sqlite3"] = sys.modules.pop("pysqlite3")
```

## Roadmap

- REST API calls as bot actions
- Webhooks for actions
- Monitoring and maintenance tools for deployed bots
