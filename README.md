
# 🌩️ FlowForge — A Minimal, Async, Persistent Workflow Engine (FastAPI)

FlowForge is a compact workflow/agent engine built using **FastAPI**, **SQLModel**, and **async node execution**.  
It demonstrates clean backend architecture, structured workflow execution, shared-state propagation, looping, branching, persistence, and WebSocket log streaming.

This project intentionally showcases:
- Clear Python code structure  
- A readable workflow engine  
- Clean API design  
- State → Transition → Loop logic  
- Async hygiene  
- A realistic example workflow: **Summarization + Refinement Pipeline**  

---

## 📦 Project Structure

flowforge/
│
├── app/
│ ├── main.py # FastAPI app: REST endpoints + WebSocket logs
│ │
│ ├── engine/ # Core workflow engine
│ │ ├── models.py # Pydantic models (NodeDef, GraphDef, RunState)
│ │ ├── registry.py # Tool registry for async node functions
│ │ └── core.py # Execution engine (state → transitions → loops)
│ │
│ ├── workflows/ # Example workflows
│ │ └── summarization_refinement.py # Summarization + Refinement agent workflow
│ │
│ ├── store/ # Persistence layer (SQLite)
│ │ └── sql.py # SQLModel storage for graphs & runs
│ │
│ └── utils/ # Utility modules
│ └── logging_config.py # Structured logging configuration
│
├── tests/
│ └── quick_run.sh # End-to-end test script
│
├── Dockerfile # Docker build
├── docker-compose.yml # Local deployment
├── requirements.txt # Dependencies
└── README.md # Documentation

yaml
Copy code

---

## 🧠 Workflow Overview: Summarization + Refinement

FlowForge implements **Option B** from the assignment:

1. **Split text into chunks**  
2. **Summarize each chunk**  
3. **Merge chunk summaries**  
4. **Refine the merged summary**  
5. **Loop until summary length ≤ target**  

### 🧩 Tools Implemented
| Tool | Description |
|------|-------------|
| `split_text` | Splits large text into fixed-size chunks |
| `summarize_chunks` | Extracts first/last sentences as a naive summary |
| `merge_summaries` | Merges all chunk summaries |
| `refine_summary` | Trims summary until it fits target length |

Each tool is asynchronous and updates the shared state.

---

## 🗺️ Graph Definition Example

{
"start": "split",
"nodes": {
"split": { "fn": "split_text", "next": "summarize" },
"summarize": { "fn": "summarize_chunks", "next": "merge" },
"merge": { "fn": "merge_summaries", "next": "refine" },
"refine": { "fn": "refine_summary", "loop_condition": "length_ok" }
}
}

shell
Copy code

### Initial State Example
{
"text": "Very long document goes here...",
"chunk_size": 500,
"target_length": 300
}

yaml
Copy code

---

## 🧩 Engine Architecture (State → Transitions → Loops)

FlowForge’s engine follows a clear execution cycle:

Load Graph → Load Initial State
→ Execute Node → Update Shared State
→ Log Result → Transition
→ Loop / Branch / Next Node
→ Repeat Until No Further Nodes
→ Mark Run Completed

markdown
Copy code

### Features
- **Async node execution** (with retries + timeout per node)
- **Shared state propagation**
- **Looping** via `loop_condition`
- **Branching** via `branches`
- **Durable runs** stored in SQLite
- **Foreground or background execution**
- **WebSocket live logs**

---

## 🌐 API Reference

### ▶ POST `/graph/create`
Registers a new workflow graph.

**Body:**
{
"start": "split",
"nodes": { ... }
}

makefile
Copy code

**Response:**
{ "graph_id": "<uuid>" }

yaml
Copy code

---

### ▶ POST `/graph/run`
Runs a workflow (foreground or background).

**Foreground run:**
{
"graph_id": "<uuid>",
"state": { ... },
"background": false
}

arduino
Copy code

Returns final state + logs.

**Background run:**
{
"graph_id": "<uuid>",
"state": { ... },
"background": true
}

makefile
Copy code

Returns:
{ "run_id": "<uuid>" }

yaml
Copy code

---

### ▶ GET `/graph/state/{run_id}`
Fetches:
- current state  
- logs  
- timestamps  
- error (if any)  
- done flag

---

### ▶ WS `/ws/logs/{run_id}`
Streams logs like:

{ "node": "summarize", "result": {...} }

makefile
Copy code

Final:

{ "done": true }

yaml
Copy code

---

## 🗃️ Persistence Layer (SQLite)

FlowForge uses **SQLModel + SQLite** to persist:

- Graph definitions  
- Run state  
- Node-by-node logs  
- Error messages  
- Timestamps  

This allows evaluators to inspect runs after the engine stops.

---

## 🧪 Testing

Run the helper script:

./tests/quick_run.sh

yaml
Copy code

It will:
- Upload the summarization graph  
- Run the workflow  
- Print final state + execution logs  

---

## 🐳 Running With Docker

### Build and start:
docker-compose up --build -d

bash
Copy code

### Visit API docs:
http://localhost:8000/docs

### Stop:
docker-compose down

yaml
Copy code

---

## 📈 Evaluation Mapping

| Requirement | How FlowForge Satisfies It |
|------------|-----------------------------|
| **Well-structured Python code** | Layered architecture: engine, workflows, store, utils |
| **Clarity of graph/engine logic** | Explicit loop, branching, transitions |
| **Clean APIs** | Only 3 main endpoints + WebSocket logs |
| **State → transitions → loops** | Fully implemented with shared-state engine |
| **Async hygiene** | All tools async; background runs; timeouts & retries |
| **Optional extras** | WebSockets, background tasks, SQLite persistence, Docker |

---

## 🚀 Future Improvements

- Graph visualization UI  
- Parallel node execution  
- Redis-backed distributed workers  
- Workflow editor GUI  
- Unit tests + CI pipeline  

---
