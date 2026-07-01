# Pipeline Builder — Interview Preparation Guide

## 1. The Big Picture — Why Build This?

### What is it?
A visual pipeline builder for AI/LLM workflows — similar to what VectorShift, LangFlow, Flowise, and Dify are building. Users design AI chains visually instead of writing code.

### Business Impact for an AI Startup:
- **Democratizes AI** — Non-engineers (product managers, marketers, analysts) can build AI workflows without writing Python
- **Faster iteration** — Developers prototype LLM chains in minutes rather than hours
- **Reduces cost** — One visual tool replaces dozens of custom scripts
- **Monetization** — Charge per execution, per user seat, or per API call to LLM providers
- **Stickiness** — Once users build pipelines, they don't leave

### Real-world competitors:
LangChain (code-first), LangFlow (visual LangChain), Flowise, VectorShift, Rivet, n8n (general automation)

---

## 2. Concepts You Must Know

### DAG (Directed Acyclic Graph)
- **Why it matters:** Data must flow forward without infinite loops. If A → B → C → A, the pipeline never terminates
- **Detection:** Kahn's algorithm (topological sort) — if you can't visit all nodes, there's a cycle
- **Interview question:** "What happens if a user creates a cycle? How do you handle it?"

### Graph Theory Basics
- **Nodes** = operations/transformations
- **Edges** = data flow between operations
- **In-degree/out-degree** = how many inputs/outputs a node has
- **Topological ordering** = valid execution sequence

### LLM Pipeline Patterns

| Pattern | Example |
|---------|---------|
| Sequential chain | Input → Prompt Template → LLM → Output |
| Parallel chains | Input → LLM-A + LLM-B → Merge → Output |
| Conditional routing | Input → Filter → (if positive → LLM-A, if negative → LLM-B) |
| RAG (Retrieval) | Input → Vector DB Search → Context + Query → LLM → Output |
| Multi-agent | Input → Agent-A → Agent-B → Consensus → Output |
| Guardrails | Input → Safety Check → LLM → Toxicity Filter → Output |

---

## 3. Tech Stack Deep-Dive

### Frontend
- **React 18** — component-based UI
- **React Flow** — graph/node rendering library (handles drag-drop, edges, zoom, pan)
- **Zustand** — lightweight state management (why not Redux? Less boilerplate for this scale)

### Backend
- **FastAPI** — async Python web framework
- **Pydantic** — request/response validation
- **Uvicorn** — ASGI server

### Key Architecture Decisions an Interviewer Might Probe
- Why Zustand over Redux or Context API?
- Why React Flow instead of building canvas from scratch?
- Why FastAPI over Flask/Django?
- How would you persist pipelines? (answer: serialize graph to JSON, store in DB)
- How would you actually *execute* a pipeline? (answer: topological sort → run nodes in order, passing outputs to inputs)

---

## 4. Questions an Interviewer Would Ask

### Architecture & Design
- How does your node abstraction work? Why did you choose this pattern?
- If I want to add a new node type, what's the minimum code I need to write?
- How would you handle node validation (e.g., LLM node requires both system + prompt connected)?
- How would you support undo/redo?
- How would you save/load pipelines?

### Frontend
- How does React Flow manage state internally?
- Explain the drag-and-drop flow from toolbar to canvas (dataTransfer API)
- How does the Text node detect variables? Walk me through the regex
- Why `useMemo` for variable extraction? What if you didn't use it?
- How would you handle 500+ nodes without UI lag? (answer: virtualization, React Flow handles this)

### Backend
- Walk me through the DAG detection algorithm. What's the time complexity? (O(V + E))
- What if the pipeline has disconnected components — is it still a DAG? (yes, each component just runs independently)
- How would you actually execute the pipeline on the backend?
- How would you handle long-running LLM calls? (answer: async execution, websockets for progress, job queue)
- How would you rate-limit API calls to OpenAI within a pipeline?

### Text Node Specifically
- How does `{{variable}}` parsing work?
- What's a "valid JavaScript variable name"? (starts with letter/$/_, followed by alphanumeric/$/_)
- What happens if a user types `{{123invalid}}`? (answer: ignored, doesn't match regex)
- How does the dynamic handle positioning work as variables are added/removed?

### Scalability & Production
- How would you deploy this? (answer: frontend on Vercel/S3+CloudFront, backend on ECS/Lambda)
- How would you add authentication?
- How would you handle concurrent pipeline executions?
- How would you add version history for pipelines?
- How would you implement real-time collaboration (multiple users editing same pipeline)?

---

## 5. Sample Workflows to Demonstrate

### Workflow 1 — Simple Q&A
```
Input → Text ("Answer this: {{question}}") → LLM → Output
```

### Workflow 2 — Summarization with Fallback
```
Input → LLM (GPT-4) → Filter (response length > 100?)
  → True: Output
  → False: LLM (retry with different prompt) → Output
```

### Workflow 3 — Multi-Model Comparison
```
Input → LLM (GPT-4) + LLM (Claude) → Merge (combine responses) → Output
```

### Workflow 4 — RAG Pipeline (Conceptual)
```
Input → API (vector search) → Merge (context + query) → Text template → LLM → Output
```

---

## 6. What the Interviewer Is Really Evaluating

| Criterion | What they look for |
|-----------|-------------------|
| Abstraction skill | Can you design a system that's easy to extend? (BaseNode pattern) |
| Frontend competence | React patterns, state management, component composition |
| System thinking | Understanding data flow, graph execution, error handling |
| CSS/Design sense | Is the UI usable and polished? |
| Backend integration | REST API design, data validation, algorithm implementation |
| Communication | Can you explain WHY you made each decision? |

---

## 7. One-Liner Answers to Keep Ready

- "I used composition over inheritance for node abstraction because React favors it"
- "Zustand because it's simpler than Redux for a single-store app with no middleware needs"
- "DAG validation is O(V+E) using Kahn's algorithm — BFS-based topological sort"
- "The Text node uses a regex with `useMemo` to avoid re-parsing on every render"
- "CORS middleware on FastAPI because frontend and backend run on different ports"
- "React Flow handles virtualization internally, so even 1000+ nodes render smoothly"

---

## 8. How to Explain Each Part in 30 Seconds

### Part 1 — Node Abstraction
"I created a `BaseNode` component that handles the shared layout — header, body, handles. Each specific node just declares its title, fields, and handle configuration. Adding a new node is ~20 lines of code instead of rewriting 50+ lines."

### Part 2 — Styling
"I built a dark-themed design using CSS custom properties for consistency. All nodes share the same visual language — rounded cards, subtle borders, colored handles. The toolbar and submit area frame the canvas cleanly."

### Part 3 — Text Node Logic
"The Text node auto-resizes using a ref on the textarea and adjusting dimensions in a `useEffect`. For variables, I use a regex to extract valid JS identifiers from `{{...}}` patterns, then render a Handle for each unique variable using `useMemo` to avoid unnecessary recalculation."

### Part 4 — Backend Integration
"The submit button reads nodes/edges from Zustand, POSTs them as JSON to FastAPI. The backend counts nodes/edges and runs Kahn's algorithm to check for cycles. The response triggers a browser alert showing the analysis."
