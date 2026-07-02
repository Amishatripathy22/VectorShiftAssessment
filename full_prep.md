# Pipeline Builder — Complete Beginner-to-Interview-Ready Guide

---

## Part A: Core Concepts (Learn These First)

---

### 1. What is a Graph?

A **graph** is a data structure with:
- **Nodes** (also called vertices) — individual items/points
- **Edges** — connections between nodes

Think of it like cities (nodes) connected by roads (edges).

**Directed Graph:** Edges have a direction. A → B means you can go from A to B, not necessarily B to A.

**Undirected Graph:** Edges go both ways.

### 2. What is a DAG?

**DAG = Directed Acyclic Graph**

- **Directed:** Each edge has a direction (like a one-way street)
- **Acyclic:** No cycles — you can never follow edges and end up back where you started

**Why pipelines must be DAGs:**
If you have nodes A → B → C → A, data would loop infinitely. A DAG guarantees that data flows forward and the pipeline terminates.

**Real-world DAG examples:**
- Git commit history (each commit points to its parent, never backward)
- Spreadsheet formulas (cell A1 depends on B1, B1 depends on C1 — never circular)
- Task scheduling (task B depends on task A finishing first)

### 3. Kahn's Algorithm (How We Detect DAGs)

```
1. Count how many incoming edges each node has (in-degree)
2. Put all nodes with 0 incoming edges in a queue
3. While queue is not empty:
   - Remove a node from queue
   - Increment visited count
   - For each neighbor of that node:
     - Decrease their in-degree by 1
     - If in-degree becomes 0, add to queue
4. If visited count == total nodes → it's a DAG
   If visited count < total nodes → there's a cycle
```

**Time complexity:** O(V + E) where V = vertices (nodes), E = edges

### 4. What is React Flow?

React Flow is a library for building node-based editors and interactive diagrams in React. It provides:
- A canvas with zoom/pan
- Draggable nodes
- Connectable handles (the dots on node edges)
- Edge rendering (the lines between nodes)
- Built-in minimap, controls, background

**You don't build the canvas from scratch** — React Flow handles all the complex canvas math, hit detection, and rendering.

### 5. What is Zustand?

Zustand is a state management library for React. Think of it as a simpler Redux.

```javascript
// Create a store
const useStore = create((set, get) => ({
  count: 0,
  increment: () => set({ count: get().count + 1 }),
}));

// Use in component
function Counter() {
  const count = useStore((state) => state.count);
  return <div>{count}</div>;
}
```

**Why Zustand over Redux?**
- No boilerplate (no actions, reducers, action creators)
- No Provider wrapper needed
- Works outside React components too
- Perfect for small-to-medium apps

**Why Zustand over React Context?**
- Context re-renders ALL consumers when ANY value changes
- Zustand only re-renders components that use the specific changed value
- Better performance with many nodes

### 6. What is FastAPI?

FastAPI is a modern Python web framework:
- Auto-generates API documentation
- Built-in request validation via Pydantic models
- Async support out of the box
- Type hints → auto-validation

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.post("/items")
def create_item(item: Item):
    return {"name": item.name, "total": item.price * 1.1}
```

### 7. React Component Composition Pattern

Our `BaseNode` uses **composition** — a parent component accepts `children` and renders them inside its layout:

```jsx
// BaseNode provides the shell
<BaseNode title="Input" handles={[...]}>
  {/* Each node provides its own content */}
  <NodeTextField label="Name" value={name} />
</BaseNode>
```

This is preferred in React over inheritance (class extending class) because:
- More flexible
- Easier to test
- Official React recommendation

### 8. Drag and Drop API

The HTML5 Drag and Drop API works like this:
1. Set `draggable` attribute on an element
2. `onDragStart` — store data about what's being dragged (`dataTransfer.setData(...)`)
3. `onDragOver` — on the drop target, prevent default to allow dropping
4. `onDrop` — read the stored data and handle the drop (`dataTransfer.getData(...)`)

In our app:
- Toolbar nodes are draggable → store node type in dataTransfer
- Canvas listens for drops → reads the type, creates a new node at that position

### 9. What is CORS?

**CORS = Cross-Origin Resource Sharing**

Browsers block requests from one origin (localhost:3000) to another (localhost:8000) by default. CORS headers tell the browser "it's okay, allow this."

We add CORS middleware to FastAPI so the React frontend can call the Python backend.

### 10. What is an LLM Pipeline?

An LLM pipeline is a sequence of steps to process data using a language model:

```
User Question → Format as Prompt → Send to GPT-4 → Parse Response → Return to User
```

A visual pipeline builder lets you design these steps by dragging and connecting nodes instead of writing code.

---

## Part B: Study Resources (Crash Course)

---

### React (if not comfortable)
- [React Official Tutorial](https://react.dev/learn) — 2 hours
- Focus on: Components, Props, State (useState), Effects (useEffect), useMemo, useRef

### React Flow
- [React Flow Docs](https://reactflow.dev/docs/introduction/) — 1 hour
- Focus on: Custom Nodes, Handles, Edges, onConnect

### Zustand
- [Zustand GitHub README](https://github.com/pmndrs/zustand) — 30 minutes
- That's literally all you need

### FastAPI
- [FastAPI First Steps](https://fastapi.tiangolo.com/tutorial/first-steps/) — 1 hour
- Focus on: Path operations, Request body, Pydantic models

### Graph Theory / DAG
- [DAG explanation (YouTube)](https://www.youtube.com/results?search_query=directed+acyclic+graph+explained) — search "DAG explained" — 15 minutes
- [Kahn's Algorithm (YouTube)](https://www.youtube.com/results?search_query=kahn%27s+algorithm+topological+sort) — 20 minutes

### CSS Custom Properties
- [MDN CSS Custom Properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) — 15 minutes

### Drag and Drop API
- [MDN Drag and Drop](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API) — 20 minutes

---

## Part C: All Interview Questions — With Answers

---

### Architecture & Design

---

**Q: How does your node abstraction work? Why did you choose this pattern?**

A: I created a `BaseNode` component that handles the shared structure every node needs — a header (icon + title), a body area for content, and dynamic Handle rendering. Each specific node (Input, LLM, Text, etc.) just passes its unique configuration:

```jsx
<BaseNode id={id} title="Input" icon="📥" handles={[...]}>
  <NodeTextField label="Name" value={name} onChange={...} />
</BaseNode>
```

I chose composition over inheritance because:
1. React officially recommends composition
2. It's more flexible — nodes can render anything in their body
3. Easy to test — BaseNode and specific nodes are independently testable
4. Adding a new node takes ~20 lines instead of ~50

---

**Q: If I want to add a new node type, what's the minimum code I need to write?**

A: Three things:

1. **Create the node file** (~15-20 lines):
```jsx
import { Position } from 'reactflow';
import { BaseNode, NodeTextField } from './BaseNode';

export const MyNode = ({ id, data }) => (
  <BaseNode id={id} title="My Node" icon="⭐" handles={[
    { type: 'target', position: Position.Left, id: `${id}-input` },
    { type: 'source', position: Position.Right, id: `${id}-output` },
  ]}>
    <NodeTextField label="Config" value="" onChange={() => {}} />
  </BaseNode>
);
```

2. **Register in ui.js** — add one line to the `nodeTypes` object:
```jsx
const nodeTypes = { ..., myNode: MyNode };
```

3. **Add to toolbar.js** — add one line:
```jsx
<DraggableNode type='myNode' label='My Node' />
```

That's it. The abstraction makes the total effort ~5 minutes and ~25 lines across 3 files.

---

**Q: How would you handle node validation (e.g., LLM node requires both system + prompt connected)?**

A: I'd add a validation layer that runs before pipeline execution:

```javascript
const validatePipeline = (nodes, edges) => {
  const errors = [];
  
  nodes.forEach(node => {
    if (node.type === 'llm') {
      const incomingEdges = edges.filter(e => e.target === node.id);
      const hasSystem = incomingEdges.some(e => e.targetHandle?.includes('system'));
      const hasPrompt = incomingEdges.some(e => e.targetHandle?.includes('prompt'));
      
      if (!hasSystem) errors.push(`${node.id}: Missing system prompt connection`);
      if (!hasPrompt) errors.push(`${node.id}: Missing user prompt connection`);
    }
  });
  
  return errors;
};
```

I'd show validation errors visually — highlight nodes with missing connections in red, show a tooltip explaining what's needed. Run validation on submit before sending to backend.

---

**Q: How would you support undo/redo?**

A: Two approaches:

**Approach 1 — State history stack (simpler):**
```javascript
// Store previous states in an array
const history = [];
const future = [];

// On every change: push current state to history
// Undo: pop from history, push current to future
// Redo: pop from future, push current to history
```

**Approach 2 — Use a library:**
Zustand has a middleware called `temporal` (zustand-middleware-undo) that handles this automatically:
```javascript
import { temporal } from 'zundo';
const useStore = create(temporal((set) => ({ ... })));
```

I'd go with Approach 2 in production — it's battle-tested and handles edge cases.

---

**Q: How would you save/load pipelines?**

A: Serialize the graph state (nodes + edges) to JSON and store it:

```javascript
// Save
const savePipeline = () => {
  const { nodes, edges } = useStore.getState();
  const pipeline = JSON.stringify({ nodes, edges, metadata: { name: 'My Pipeline', version: 1 } });
  // POST to backend → store in PostgreSQL/MongoDB
};

// Load
const loadPipeline = async (id) => {
  const response = await fetch(`/pipelines/${id}`);
  const { nodes, edges } = await response.json();
  useStore.setState({ nodes, edges });
};
```

For the database, I'd store:
- Pipeline ID, name, owner
- `graph_json` — the serialized nodes/edges
- Created/updated timestamps
- Version number for history

---

### Frontend Questions

---

**Q: How does React Flow manage state internally?**

A: React Flow uses a controlled component pattern. WE own the state (nodes and edges arrays in Zustand), and React Flow just renders whatever we pass:

```jsx
<ReactFlow
  nodes={nodes}           // We control this
  edges={edges}           // We control this
  onNodesChange={...}     // RF tells us what changed, we update our state
  onEdgesChange={...}     // Same pattern
  onConnect={...}         // New edge created, we add to our state
/>
```

React Flow internally uses:
- An internal store for viewport (zoom, pan position)
- DOM measurements for node dimensions
- Hit testing for drag/drop/connection detection
- SVG for edge rendering

---

**Q: Explain the drag-and-drop flow from toolbar to canvas.**

A: Step by step:

1. **Toolbar** — `DraggableNode` has `draggable` attribute. On `dragStart`, we store the node type in `dataTransfer`:
   ```javascript
   event.dataTransfer.setData('application/reactflow', JSON.stringify({ nodeType: 'llm' }));
   ```

2. **Canvas** — `onDragOver` calls `event.preventDefault()` to allow dropping.

3. **Drop** — `onDrop` fires:
   - Read the node type from `dataTransfer.getData(...)`
   - Use `reactFlowInstance.project()` to convert screen coordinates to canvas coordinates
   - Generate a unique node ID via `getNodeID(type)`
   - Create a node object with `{ id, type, position, data }`
   - Call `addNode()` to append to the Zustand store
   - React Flow re-renders with the new node on canvas

---

**Q: How does the Text node detect variables? Walk me through the regex.**

A: The regex: `/\{\{\s*([a-zA-Z_$][a-zA-Z0-9_$]*)\s*\}\}/g`

Breaking it down:
- `\{\{` — matches literal `{{`
- `\s*` — optional whitespace (so `{{ input }}` works)
- `([a-zA-Z_$][a-zA-Z0-9_$]*)` — captures a valid JS identifier:
  - Must start with letter, underscore, or dollar sign
  - Followed by any number of letters, digits, underscore, or dollar sign
- `\s*` — optional trailing whitespace
- `\}\}` — matches literal `}}`
- `g` flag — find ALL matches, not just the first

Examples:
- `{{input}}` → captures `input` ✓
- `{{ user_name }}` → captures `user_name` ✓
- `{{$price}}` → captures `$price` ✓
- `{{123bad}}` → no match (starts with digit) ✗
- `{{hello world}}` → no match (space in name) ✗

---

**Q: Why `useMemo` for variable extraction? What if you didn't use it?**

A: `useMemo` caches the computed result and only recalculates when `currText` changes.

Without `useMemo`:
```javascript
// This runs on EVERY render (parent re-render, unrelated state change, etc.)
const variables = extractVariables(currText);
```

With `useMemo`:
```javascript
// Only recalculates when currText actually changes
const variables = useMemo(() => extractVariables(currText), [currText]);
```

For this specific case, the regex is cheap, so performance impact is minimal. But it's good practice because:
1. If the regex were expensive, it prevents unnecessary work
2. It gives React a stable reference — so child components (Handles) don't re-render unnecessarily
3. Shows the interviewer you understand React optimization

---

**Q: How would you handle 500+ nodes without UI lag?**

A: React Flow already handles this through:
1. **Viewport culling** — Only renders nodes visible in the current viewport
2. **Virtualization** — Nodes outside the viewport are unmounted from DOM

If I still hit performance issues:
- Use `React.memo` on custom node components to prevent unnecessary re-renders
- Use `nodesDraggable={false}` for read-only mode
- Debounce state updates (batch rapid changes)
- Consider Web Workers for heavy computation (like DAG validation on large graphs)

---

### Backend Questions

---

**Q: Walk me through the DAG detection algorithm. What's the time complexity?**

A: We use Kahn's algorithm:

```python
def is_dag(nodes, edges):
    # 1. Build adjacency list and count in-degrees
    adj = {node_id: [] for node in nodes}
    in_degree = {node_id: 0 for node in nodes}
    
    for edge in edges:
        adj[edge.source].append(edge.target)
        in_degree[edge.target] += 1
    
    # 2. Start with all nodes that have no incoming edges
    queue = [n for n in in_degree if in_degree[n] == 0]
    visited = 0
    
    # 3. Process queue — remove each node, decrease neighbors' in-degree
    while queue:
        current = queue.pop(0)
        visited += 1
        for neighbor in adj[current]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)
    
    # 4. If we visited all nodes, no cycle exists
    return visited == len(nodes)
```

**Time complexity: O(V + E)**
- Building adjacency list: O(E)
- Processing all nodes: O(V)
- Decreasing in-degrees: O(E) total across all iterations

**Space complexity: O(V + E)** for the adjacency list

---

**Q: What if the pipeline has disconnected components — is it still a DAG?**

A: Yes! A disconnected graph with no cycles is still a DAG. Kahn's algorithm handles this naturally because:
- Each disconnected component will have at least one node with in-degree 0
- The algorithm processes all components independently
- As long as no component contains a cycle, the total visited count equals total nodes

In practice, disconnected components would execute independently (in parallel if we wanted).

---

**Q: How would you actually execute the pipeline on the backend?**

A: I'd use topological sort to determine execution order:

```python
async def execute_pipeline(nodes, edges):
    # 1. Topological sort to get execution order
    execution_order = topological_sort(nodes, edges)
    
    # 2. Execute each node, passing outputs to connected inputs
    results = {}
    
    for node_id in execution_order:
        node = get_node(node_id)
        
        # Gather inputs from connected source nodes
        inputs = {}
        for edge in edges:
            if edge.target == node_id:
                inputs[edge.targetHandle] = results[edge.source][edge.sourceHandle]
        
        # Execute based on node type
        if node.type == 'llm':
            results[node_id] = await call_openai(inputs['system'], inputs['prompt'])
        elif node.type == 'text':
            results[node_id] = interpolate_template(node.data.text, inputs)
        elif node.type == 'filter':
            results[node_id] = apply_filter(inputs, node.data.condition)
        # ... etc
    
    return results
```

Key considerations:
- Nodes with no dependencies can run in parallel (same topological level)
- LLM calls are async and can take seconds — need timeout handling
- Store intermediate results for debugging

---

**Q: How would you handle long-running LLM calls?**

A: Multiple strategies:

1. **Async execution with job queue:**
   ```
   POST /pipelines/execute → returns { job_id: "abc123" }
   GET /pipelines/jobs/abc123 → returns { status: "running", progress: "3/7 nodes" }
   ```

2. **WebSocket for real-time updates:**
   ```javascript
   const ws = new WebSocket('ws://localhost:8000/ws/pipeline/abc123');
   ws.onmessage = (event) => {
     const update = JSON.parse(event.data);
     // update.node_id, update.status, update.result
     highlightNodeOnCanvas(update.node_id, update.status);
   };
   ```

3. **Server-Sent Events (SSE) for streaming:**
   - LLM responses stream token-by-token
   - Show partial results as they arrive

4. **Timeout + retry:**
   - Set per-node timeout (e.g., 30s for LLM calls)
   - Retry with exponential backoff on failure
   - Dead-letter queue for permanently failed nodes

---

**Q: How would you rate-limit API calls to OpenAI within a pipeline?**

A: Multiple layers:

1. **Token bucket per user:**
   ```python
   from asyncio import Semaphore
   
   user_semaphores = {}  # user_id → Semaphore(max_concurrent=5)
   
   async def call_llm(user_id, prompt):
       sem = user_semaphores.setdefault(user_id, Semaphore(5))
       async with sem:
           return await openai.chat.completions.create(...)
   ```

2. **Global rate limiter respecting OpenAI's limits:**
   - Track tokens per minute (TPM) and requests per minute (RPM)
   - Queue requests when approaching limits
   - Use `aiolimiter` or custom sliding window

3. **Cost controls:**
   - Set per-user monthly budget
   - Estimate token count before sending
   - Alert at 80% budget usage

---

### Text Node Questions

---

**Q: How does `{{variable}}` parsing work?**

A: Every time the text changes:

1. `currText` state updates via `onChange`
2. `useMemo` triggers because `currText` is in its dependency array
3. Regex scans the entire text for `{{...}}` patterns
4. Extracts valid JS variable names (ignoring duplicates)
5. Returns an array like `['input', 'context', 'query']`
6. React renders one `<Handle>` component per variable on the left side
7. Handle position is calculated: `top = (index + 1) / (totalVars + 1) * 100%` — evenly distributed vertically

---

**Q: What's a "valid JavaScript variable name"?**

A: Per the ECMAScript spec:
- **Must start with:** letter (a-z, A-Z), underscore (_), or dollar sign ($)
- **Followed by:** any combination of letters, digits (0-9), underscores, or dollar signs
- **Cannot be** a reserved word (but we don't check this since it's just a label)

Valid: `input`, `_private`, `$price`, `userName2`, `__proto`
Invalid: `123abc`, `my-var` (hyphen), `hello world` (space), `class` (reserved, but we allow it for simplicity)

---

**Q: What happens if a user types `{{123invalid}}`?**

A: Nothing — it's ignored. The regex `[a-zA-Z_$]` requires the first character to be a letter, underscore, or dollar sign. Since `1` doesn't match, the regex engine skips this match entirely. No handle is created.

---

**Q: How does the dynamic handle positioning work as variables are added/removed?**

A: Handles are positioned using percentage-based `top` values:

```jsx
style={{ top: `${((index + 1) / (variables.length + 1)) * 100}%` }}
```

With 1 variable: handle at 50%
With 2 variables: handles at 33% and 66%
With 3 variables: handles at 25%, 50%, 75%

When a variable is added/removed:
1. `useMemo` recalculates the variables array
2. React re-renders the handles with new positions
3. React Flow automatically updates any connected edges to follow the new handle positions

This means existing connections stay intact even as handles move — React Flow handles edge re-routing.

---

### Scalability & Production Questions

---

**Q: How would you deploy this?**

A: 
**Frontend:**
- Build static files: `npm run build`
- Host on S3 + CloudFront (or Vercel/Netlify)
- CDN for global low-latency access
- Cost: essentially free at moderate scale

**Backend:**
- Container (Docker) deployed on ECS Fargate or EKS
- Auto-scaling group behind ALB
- Or: Lambda + API Gateway for serverless (if requests are bursty)

**Database:**
- PostgreSQL (RDS) for pipeline metadata
- Redis for session/cache
- S3 for large pipeline exports

**CI/CD:**
- GitHub Actions → build → test → deploy
- Staging environment for QA

---

**Q: How would you add authentication?**

A: 
1. **JWT-based auth:**
   - Frontend: Login form → POST /auth/login → receive JWT → store in httpOnly cookie
   - Backend: Middleware validates JWT on every request
   - Use refresh tokens for long sessions

2. **OAuth2 (Google/GitHub login):**
   - Redirect to provider → callback with code → exchange for token
   - Map external identity to internal user

3. **FastAPI implementation:**
   ```python
   from fastapi import Depends
   from fastapi.security import HTTPBearer
   
   security = HTTPBearer()
   
   async def get_current_user(token = Depends(security)):
       payload = jwt.decode(token.credentials, SECRET_KEY)
       return payload["user_id"]
   
   @app.post("/pipelines/parse")
   async def parse(pipeline: PipelineRequest, user_id = Depends(get_current_user)):
       ...
   ```

---

**Q: How would you handle concurrent pipeline executions?**

A: 
1. **Job queue (Celery/Redis or SQS):**
   - Submit creates a job → worker picks it up → executes asynchronously
   - Multiple workers process jobs in parallel
   - Each user can have N concurrent executions (configurable limit)

2. **Isolation:**
   - Each execution runs in its own context (no shared mutable state)
   - Results stored per-execution-ID

3. **Resource management:**
   - Track active executions per user
   - Queue excess requests (FIFO)
   - Timeout long-running executions (e.g., 5-minute max)

---

**Q: How would you add version history for pipelines?**

A: 
```sql
CREATE TABLE pipeline_versions (
    id UUID PRIMARY KEY,
    pipeline_id UUID REFERENCES pipelines(id),
    version INT,
    graph_json JSONB,        -- nodes + edges snapshot
    created_at TIMESTAMP,
    created_by VARCHAR,
    commit_message TEXT
);
```

On every save:
- Increment version number
- Store full snapshot of nodes + edges
- Allow user to view diff between versions
- "Restore" button loads old version into current state

Frontend could show a version timeline (like Git log) with the ability to preview any version.

---

**Q: How would you implement real-time collaboration?**

A: Use CRDTs (Conflict-free Replicated Data Types) or OT (Operational Transform):

1. **WebSocket connection** per user per pipeline
2. **Broadcast operations** (not full state):
   - "User A moved node-3 to position (100, 200)"
   - "User B added edge from node-1 to node-2"
3. **Conflict resolution:**
   - Last-write-wins for node positions
   - CRDTs for concurrent adds/deletes
4. **Presence indicators:**
   - Show cursors/avatars of other users
   - Highlight nodes being edited by others

Libraries to consider: Yjs, Automerge, or Liveblocks (hosted solution).

---

## Part D: How to Structure Your Interview Answers

---

### The STAR Method for Technical Answers

**Situation:** "The problem was..."
**Task:** "I needed to..."
**Action:** "I chose to... because..."
**Result:** "This gave us..."

### Example:

> **Q: Why did you use Zustand?**
>
> "The app needed shared state between the toolbar, canvas, and submit button (nodes/edges). I evaluated three options: React Context, Redux, and Zustand. Context causes unnecessary re-renders for frequently changing data like node positions. Redux is powerful but adds significant boilerplate for a project of this size. Zustand gave me a simple API with selective re-rendering — components only update when their specific slice of state changes. The result is clean code with good performance."

---

## Part E: Quick Self-Test

Before the interview, make sure you can answer these without looking:

1. ☐ What's a DAG and why must pipelines be DAGs?
2. ☐ How does Kahn's algorithm work? (explain in 30 seconds)
3. ☐ What does `BaseNode` do and why does it exist?
4. ☐ How does drag-and-drop work (toolbar → canvas)?
5. ☐ How does the Text node regex work?
6. ☐ Why Zustand over Redux?
7. ☐ What does the submit button actually send to the backend?
8. ☐ How does the backend validate if it's a DAG?
9. ☐ What is CORS and why do we need it?
10. ☐ How would you add a new node type? (3 steps)
