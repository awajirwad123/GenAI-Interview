# LangChain, LangGraph & Agents - Essentials

## LangChain Core Concepts

### 1. Chains
```python
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate

prompt = PromptTemplate(template="Summarize: {text}", input_variables=["text"])
chain = LLMChain(llm=llm, prompt=prompt)
result = chain.run(text="long document...")
```

**Key chains:**
- **LLMChain**: Basic prompt + LLM
- **SequentialChain**: Chain outputs together
- **RouterChain**: Route to different chains based on input

### 2. Memory
```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory()
memory.save_context({"input": "Hi"}, {"output": "Hello!"})
memory.load_memory_variables({})  # Get history
```

**Types:**
- **BufferMemory**: Store all messages
- **WindowMemory**: Last N messages
- **SummaryMemory**: Summarize old messages
- **VectorStoreMemory**: Semantic retrieval of relevant history

### 3. Agents & Tools
```python
from langchain.agents import create_react_agent, Tool

tools = [
    Tool(name="Calculator", func=calculator, description="For math"),
    Tool(name="Search", func=search_web, description="For web search")
]

agent = create_react_agent(llm=llm, tools=tools, prompt=prompt_template)
result = agent.invoke({"input": "What's 25 * 17 and what's the weather?"})
```

## LangGraph - State Machines for Agents

### Basic Structure
```python
from langgraph.graph import StateGraph

# Define state
class State(TypedDict):
    messages: List[str]
    next_step: str

# Build graph
graph = StateGraph(State)
graph.add_node("retrieve", retrieve_docs)
graph.add_node("generate", generate_answer)
graph.add_edge("retrieve", "generate")
graph.set_entry_point("retrieve")

app = graph.compile()
result = app.invoke({"messages": ["user query"]})
```

**When to use LangGraph:**
- ✅ Complex multi-step workflows
- ✅ Conditional branching (if doc quality low, retry)
- ✅ Cycles (iterate until condition met)
- ✅ Human-in-loop checkpoints
- ❌ Simple linear chains (use LangChain)

## Agent Patterns

### 1. ReAct Agent
```
Thought: I need weather info
Action: search_weather("Paris")
Observation: 18°C, sunny
Thought: Now I can answer
Final Answer: Paris is 18°C and sunny
```

### 2. Plan-and-Execute
```python
# Step 1: Plan
plan = planner.invoke("Complex query")
# Output: ["Step 1: ...", "Step 2: ...", "Step 3: ..."]

# Step 2: Execute each step
results = []
for step in plan:
    result = executor.invoke(step)
    results.append(result)

# Step 3: Synthesize
final = synthesizer.invoke({"plan": plan, "results": results})
```

### 3. Multi-Agent Collaboration
```python
researcher = Agent(tools=[search, scrape], role="researcher")
analyst = Agent(tools=[calculate, chart], role="analyst")
writer = Agent(tools=[grammar_check], role="writer")

# Workflow
data = researcher.run("Find Q4 revenue data")
insights = analyst.run(f"Analyze: {data}")
report = writer.run(f"Write report on: {insights}")
```

## Production Considerations

**Cost management:**
- Tool calls add 20-50% cost vs direct prompts
- Agent loops can spiral (set max_iterations=5)
- Cache tool results when possible

**Reliability:**
- Agents fail ~10-20% of the time
- Implement fallbacks and retries
- Log full traces for debugging

**Latency:**
- Each tool call adds LLM inference time
- Parallelize independent tool calls
- Consider smaller models for tool selection

## Interview-Critical Points

**Q: LangChain vs direct API calls?**
- LangChain: Rapid prototyping, standardization
- Direct: Full control, less abstraction overhead
- Production: Often migrate from LangChain to custom

**Q: When NOT to use agents?**
- Deterministic workflows → use fixed chains
- Low-latency requirements → too slow
- Cost-sensitive → agents use 2-5× more tokens

**Q: LangChain vs LangGraph?**
- LangChain: Linear chains, simple agents
- LangGraph: Complex workflows, state management, cycles
- Use both: LangChain components in LangGraph nodes
