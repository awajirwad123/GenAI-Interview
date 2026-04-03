# LangChain & LangGraph - Interview Questions

### Q1: When would you use LangChain vs writing custom code?

**LangChain pros:**
- Fast prototyping (hours vs days)
- Standardized interfaces (swap LLMs easily)
- Built-in RAG, memory, agents

**Custom code pros:**
- Full control, less abstraction
- Better performance (no framework overhead)
- Easier debugging

**Production reality**: Start with LangChain (prototype), migrate to custom for production. We went from LangChain to custom after 3 months.

---

### Q2: Explain how ReAct agents work and their limitations.

**How they work:**
```
Thought: I need current weather
Action: search_weather("Paris")
Observation: 18°C
Thought: Got the info
Answer: 18°C in Paris
```

**Limitations:**
1. **Loops**: May cycle between same tools (set max_iterations)
2. **Tool selection errors**: Picks wrong tool 15-20% of time
3. **Context length**: Long traces exceed context window
4. **Cost**: 3-5× more tokens than direct prompt
5. **Latency**: Each loop adds 1-2s

**Mitigations:**
- Better tool descriptions
- Few-shot examples in prompt
- Fallback to fixed workflow if agent loops

---

### Q3: Design a multi-step agent for research + analysis tasks.

```python
from langgraph.graph import StateGraph

class ResearchState(TypedDict):
    query: str
    search_results: List[str]
    analysis: str
    final_report: str
    quality_check: bool

# Node: Search
def search_node(state):
    results = search_web(state["query"])
    return {"search_results": results}

# Node: Analyze
def analyze_node(state):
    analysis = llm.analyze(state["search_results"])
    return {"analysis": analysis}

# Node: Quality check
def quality_node(state):
    quality = llm.check_quality(state["analysis"])
    return {"quality_check": quality}

# Node: Generate report
def report_node(state):
    report = llm.generate_report(state["analysis"])
    return {"final_report": report}

# Build graph
graph = StateGraph(ResearchState)
graph.add_node("search", search_node)
graph.add_node("analyze", analyze_node)
graph.add_node("quality", quality_node)
graph.add_node("report", report_node)

# Conditional edge: if quality low, redo search
graph.add_conditional_edges(
    "quality",
    lambda state: "search" if not state["quality_check"] else "report"
)

graph.set_entry_point("search")
app = graph.compile()
```

**Key features:**
- Cycles for retry logic
- State management across nodes
- Conditional branching

---

### Q4: How do you debug failing agents?

**Debugging process:**

1. **Enable tracing**
```python
from langsmith import Client

client = Client()
with trace_run("agent_debug"):
    result = agent.invoke(input)
# View full trace in LangSmith UI
```

2. **Log tool calls**
```python
def logged_tool(func):
    def wrapper(*args, **kwargs):
        print(f"Tool called: {func.__name__} with {args}")
        result = func(*args, **kwargs)
        print(f"Tool result: {result}")
        return result
    return wrapper
```

3. **Check common failures**
- Agent loops (check iteration count)
- Wrong tool selection (improve descriptions)
- Context overflow (summarize history)
- Tool errors (validate inputs)

4. **Simplify and isolate**
```python
# Test tool in isolation
result = search_tool("test query")

# Test with fixed workflow (no agent)
result = sequential_chain.run(input)
```

**Production monitoring:**
```python
metrics = {
    "success_rate": 0.85,
    "avg_iterations": 2.3,
    "avg_latency": 3.5s,
    "tool_error_rate": 0.05
}
```

---

### Q5: Scenario: Agent costs are 5× higher than expected. Root cause and fix?

**Investigation:**

```python
# Analyze agent traces
traces = get_traces(agent_id, last_24h)

avg_iterations = mean([t.iterations for t in traces])  # 8.3
avg_tokens = mean([t.total_tokens for t in traces])  # 12,450

# Root causes identified:
# 1. Looping (8 iterations vs expected 2-3)
# 2. Long context (12K tokens vs expected 3K)
```

**Fixes:**

1. **Limit iterations**
```python
agent = create_agent(
    tools=tools,
    max_iterations=5,  # Hard limit
    early_stopping_method="generate"  # Force answer
)
```

2. **Compress context**
```python
# Summarize conversation history
if len(messages) > 10:
    summary = llm.summarize(messages[:-5])
    messages = [summary] + messages[-5:]
```

3. **Better tool descriptions**
```python
# Bad: description="Search tool"
# Good: description="Search for current web info. Use for: recent events, news, current data. Don't use for: math, general knowledge."
```

4. **Fallback to fixed workflow**
```python
if agent_cost > threshold:
    # Use deterministic chain instead
    result = fixed_chain.run(input)
else:
    result = agent.run(input)
```

**Results:**
- Iterations: 8.3 → 2.8
- Tokens: 12K → 4K
- Cost: 5× → 1.5× expected
