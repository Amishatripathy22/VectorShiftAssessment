# Pipeline Builder — Project Walkthrough Script

## Recommended Time: 15–20 minutes

| Section | Time | What You Cover |
|---------|------|----------------|
| 1. Problem Statement & Why | 2 min | What and why |
| 2. High-Level Architecture | 2 min | System overview |
| 3. Node Abstraction (Part 1) | 3 min | Code design |
| 4. Styling (Part 2) | 1 min | Quick mention |
| 5. Text Node Logic (Part 3) | 3 min | Regex + dynamic handles |
| 6. Backend Integration (Part 4) | 3 min | API + DAG algorithm |
| 7. Live Demo | 3-4 min | Build a pipeline, submit |
| 8. Wrap-up / Extensions | 1-2 min | What I'd do next |

---

## THE SCRIPT

---

### 1. Problem Statement & Why (2 min)

> "The goal of this project is to build a visual pipeline builder for AI/LLM workflows.
>
> Think of tools like LangFlow or VectorShift — where non-technical users can design AI chains by dragging nodes onto a canvas and connecting them, instead of writing code.
>
> The business value is clear: it democratizes AI. A product manager can build a summarization pipeline without writing Python. It speeds up prototyping for engineers too — you can test different LLM configurations visually before committing to code.
>
> The assignment had four parts:
> 1. Create a node abstraction to make adding new nodes easy
> 2. Style the application into a polished design
> 3. Add dynamic behavior to the Text node — auto-resizing and variable detection
> 4. Connect the frontend to a FastAPI backend that validates the pipeline structure"

---

### 2. High-Level Architecture (2 min)

> "Let me show you the architecture."

Draw or describe:

```
┌─────────────────────────────────────────────────┐
│                  FRONTEND (React)                 │
│                                                   │
│  ┌──────────┐   ┌────────────┐   ┌───────────┐  │
│  │ Toolbar  │   │   Canvas   │   │  Submit   │  │
│  │(drag src)│   │(React Flow)│   │  Button   │  │
│  └──────────┘   └────────────┘   └───────────┘  │
│                        │                          │
│               ┌────────────────┐                  │
│               │  Zustand Store │                  │
│               │ (nodes, edges) │                  │
│               └────────────────┘                  │
└───────────────────────┬──────────────────────────┘
                        │ POST /pipelines/parse
                        │ { nodes: [...], edges: [...] }
                        ▼
┌─────────────────────────────────────────────────┐
│               BACKEND (FastAPI)                   │
│                                                   │
│  • Count nodes & edges                           │
│  • DAG validation (Kahn's algorithm)             │
│  • Return { num_nodes, num_edges, is_dag }       │
└─────────────────────────────────────────────────┘
```

> "The frontend uses React with React Flow for the canvas, and Zustand for state management. The backend is FastAPI with a single endpoint that takes the graph and validates it.
>
> I chose Zustand over Redux because there's no need for middleware or complex action patterns here — Zustand gives selective re-rendering with minimal boilerplate.
>
> React Flow was the natural choice because it gives us drag-drop, zooming, panning, edge routing, and handle connections out of the box."

---

### 3. Node Abstraction — Part 1 (3 min)

> "The original code had four node files that shared a lot of structure — a container div, a title, some form fields, and handles. If you wanted to add a new node, you'd copy-paste an existing file and modify it. That doesn't scale."

**Open `BaseNode.js`:**

> "I created a `BaseNode` component using React's composition pattern. It accepts:
> - `title` and `icon` for the header
> - `children` for the body content (each node provides its own fields)
> - `handles` array — a declarative way to define input/output connection points
>
> I also created helper components — `NodeTextField`, `NodeSelectField`, `NodeTextArea` — so common form patterns are one-liners."

**Open `inputNode.js` as an example:**

> "Now look how clean a node definition is. It's ~20 lines. I declare what fields I need and what handles exist. The abstraction handles all layout, styling, and handle rendering.
>
> To prove this scales, I created 5 new nodes in just a few minutes — Filter, Merge, Timer, Note, and API — each showcasing different handle configurations. Filter has conditional branching with two outputs. Note has zero handles. API has both response and error outputs."

---

### 4. Styling — Part 2 (1 min)

> "The original app had no styling — just borders on divs. I built a dark-themed design using CSS custom properties for consistency — all colors, borders, shadows, and radii are defined as variables so the entire theme can be changed in one place.
>
> Key decisions:
> - Dark theme because it's standard for developer tools
> - All nodes share the same visual card structure from BaseNode
> - Handles have hover animations so users know they're interactive
> - The Note node has a distinct warm color to differentiate it as non-functional"

---

### 5. Text Node Logic — Part 3 (3 min)

**Open `textNode.js`:**

> "The Text node has two special behaviors.
>
> **First — dynamic resizing.** I attach a ref to the textarea and use a `useEffect` that fires when `currText` changes. It measures the textarea's `scrollHeight` and adjusts the node's dimensions. Width also grows based on text length, capped at 400px. This means the node grows as the user types, so they always see their full text.
>
> **Second — variable detection.** I use a regex inside `useMemo`:"

```
/\{\{\s*([a-zA-Z_$][a-zA-Z0-9_$]*)\s*\}\}/g
```

> "This finds any text wrapped in double curly braces that contains a valid JavaScript variable name. So `{{input}}` creates a variable, but `{{123bad}}` is ignored.
>
> For each unique variable found, I render a Handle on the left side of the node. The handles are positioned evenly using percentage-based math. Each handle has a label showing the variable name.
>
> I used `useMemo` because we only need to re-run the regex when text actually changes — not on every render. This also gives React a stable array reference, preventing unnecessary Handle re-renders."

---

### 6. Backend Integration — Part 4 (3 min)

**Open `submit.js`:**

> "The Submit button reads nodes and edges directly from Zustand, then POSTs them as JSON to the backend at `/pipelines/parse`.
>
> When the response comes back, I show a browser alert with the analysis: number of nodes, number of edges, and whether it's a valid DAG."

**Open `main.py`:**

> "The backend uses FastAPI with Pydantic for request validation. The key logic is the `is_dag` function.
>
> I'm using Kahn's algorithm for cycle detection:
> 1. Build an adjacency list and count in-degrees for all nodes
> 2. Start with all nodes that have zero incoming edges — these are entry points
> 3. Process each node: for every neighbor, decrease their in-degree. If it hits zero, add to queue
> 4. If we visit all nodes, there's no cycle — it's a DAG. If some nodes remain unvisited, they're trapped in a cycle.
>
> Time complexity is O(V + E) — linear in the size of the graph. This scales fine even for hundreds of nodes.
>
> I also added CORS middleware because the frontend runs on port 3000 and backend on port 8000 — different origins. Without CORS, the browser blocks the request."

---

### 7. Live Demo (3-4 min)

> "Let me build a real pipeline to show this working."

**Demo 1 — Simple RAG-like chain:**

1. Drag an **Input** node → name it "user_query"
2. Drag a **Text** node → type: `"Given this context: {{context}}\n\nAnswer: {{question}}"`
   - Show: "See how two handles appeared — 'context' and 'question'"
3. Drag an **API** node → set URL to a vector DB endpoint, method GET
4. Drag an **LLM** node → select GPT-4
5. Drag an **Output** node → name it "answer"
6. Connect: Input → Text's "question" handle, API response → Text's "context" handle, Text output → LLM prompt, LLM response → Output
7. Click Submit → "Pipeline Analysis: 5 nodes, 4 edges, is DAG: Yes ✓"

> "This represents a RAG pipeline — Retrieval Augmented Generation — where we fetch context from a database and combine it with the user's question before sending to the LLM."

**Demo 2 — Creating a cycle (to show DAG detection):**

1. Add two nodes and connect A → B → A (a loop)
2. Click Submit → "is DAG: No ✗"

> "The backend correctly identifies the cycle. In a production system, we'd block execution and highlight the problematic edges."

**Demo 3 — Complex multi-branch:**

1. Input → Filter (condition: "length > 100")
2. Filter TRUE → LLM (GPT-4) for summarization
3. Filter FALSE → directly to Output (short text doesn't need summarization)
4. LLM → Merge with the other branch → Output

> "This shows conditional routing — we only call the LLM when the input is long enough to need summarization. This saves API costs."

---

### 8. Wrap-up / Extensions (1-2 min)

> "If I had more time, here's what I'd add:
>
> 1. **Actual pipeline execution** — topological sort the nodes, run each one, pass data forward
> 2. **Validation** — highlight unconnected required handles in red before submission
> 3. **Save/load** — serialize to JSON, store in a database, let users manage multiple pipelines
> 4. **Real-time execution feedback** — WebSocket to stream node-by-node progress
> 5. **Undo/redo** — using Zustand's temporal middleware
>
> The abstraction I built makes the system highly extensible. Adding a new node type takes 3 files and about 20 lines of code. The separation of concerns — BaseNode for layout, individual nodes for logic, Zustand for state, FastAPI for validation — means each piece can evolve independently."

---

## TIPS FOR DELIVERY

1. **Don't read code line-by-line.** Point at key sections and explain the concept.
2. **Use the demo to impress.** Building a pipeline live is more compelling than slides.
3. **Anticipate follow-ups.** After each section, pause briefly. If they ask "why not X?", have a reason.
4. **Own what you don't know.** If asked about something you haven't implemented, say "I'd approach it like..." rather than guessing.
5. **Keep energy up.** This is YOUR project. Show enthusiasm for the design decisions.

---

## IS THE FULL PREP DOC SUFFICIENT?

**For understanding the code: Yes, mostly.** The full prep doc explains what each piece does and why. But you should also:

1. **Actually read each file** — Open every .js and .py file. Trace the data flow manually:
   - User drags → `draggableNode.js` stores type → `ui.js` handles drop → `store.js` adds node → React Flow renders it
   - User connects → `store.js` onConnect adds edge → React Flow renders edge
   - User clicks submit → `submit.js` reads store → fetches backend → shows alert

2. **Run the app yourself** and play with it for 10 minutes. Build different pipelines. Break things intentionally (create cycles, disconnect nodes). This builds intuition no document can give you.

3. **Modify something small** — Change a node's fields, add a 6th new node. If you can do this in 5 minutes without looking at the docs, you truly understand the abstraction.

**For the interview itself:** Combine the full prep doc (for Q&A) with this walkthrough script (for the demo portion). Practice the walkthrough out loud 2-3 times. Time yourself — it should be 15-20 minutes.
