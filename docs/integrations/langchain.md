# LangChain - Quick Reference

Quick reference for monitoring LangChain applications with WhiteBoxXAI.

## Installation

```bash
pip install whitebox-xai-sdk langchain openai
```

## Basic Setup

```python
from langchain.chains import LLMChain
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.langchain import LangChainMonitor

# Initialize
client = WhiteBoxXAI(api_key="your-api-key")
monitor = LangChainMonitor(
    client=client,
    application_name="my_app",
    track_tokens=True,
    track_cost=True
)
monitor.register_application(name="My App", version="1.0.0")
```

## Quick Patterns

### Callback Handler (Recommended)

```python
# Create callback once
callback = monitor.create_callback_handler()

# Use with any component
chain.run(input="...", callbacks=[callback])
agent.run(input="...", callbacks=[callback])
```

### Wrap Chain

```python
from whiteboxxai.integrations.langchain import wrap_langchain_chain

wrapped_chain = wrap_langchain_chain(chain, monitor)
result = wrapped_chain.run(input="...")  # Auto-logged
```

### Manual Logging

```python
monitor.log_chain_execution(
    chain_name="my_chain",
    inputs={"question": "What is AI?"},
    outputs={"answer": "AI is..."},
    execution_time=1.5
)
```

## Common Use Cases

### Simple LLM Chain

```python
llm = OpenAI(temperature=0.7)
prompt = PromptTemplate(
    input_variables=["question"],
    template="Answer: {question}"
)
chain = LLMChain(llm=llm, prompt=prompt)

callback = monitor.create_callback_handler()
result = chain.run(question="What is AI?", callbacks=[callback])
```

### Sequential Chain

```python
from langchain.chains import SequentialChain

chain1 = LLMChain(llm=llm, prompt=prompt1, output_key="topic")
chain2 = LLMChain(llm=llm, prompt=prompt2, output_key="essay")

overall = SequentialChain(
    chains=[chain1, chain2],
    input_variables=["subject"],
    output_variables=["topic", "essay"]
)

wrapped = wrap_langchain_chain(overall, monitor)
result = wrapped({"subject": "space"})
```

### Agent with Tools

```python
from langchain.agents import initialize_agent, AgentType, Tool

tools = [
    Tool(name="Search", func=search_func, description="Search tool")
]

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION
)

callback = monitor.create_callback_handler()
result = agent.run("Find info about AI", callbacks=[callback])
```

### RAG Pipeline

```python
from langchain.chains import RetrievalQA
from langchain.vectorstores import FAISS

qa = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectorstore.as_retriever()
)

callback = monitor.create_callback_handler()
result = qa.run("Question?", callbacks=[callback])

# Log retrieval details
monitor.log_rag_retrieval(
    query="Question?",
    documents=docs,
    num_retrieved=len(docs),
    retrieval_time=0.5
)
```

### Conversational Agent

```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory(memory_key="chat_history")

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
    memory=memory
)

callback = monitor.create_callback_handler()
result1 = agent.run("My name is Alice", callbacks=[callback])
result2 = agent.run("What's my name?", callbacks=[callback])
```

## Monitoring Methods

| Method | Purpose | Example |
|--------|---------|---------|
| `register_application()` | Register app | `monitor.register_application(name="App")` |
| `create_callback_handler()` | Create callback | `callback = monitor.create_callback_handler()` |
| `log_chain_execution()` | Log chain run | `monitor.log_chain_execution(...)` |
| `log_agent_execution()` | Log agent run | `monitor.log_agent_execution(...)` |
| `log_llm_call()` | Log LLM call | `monitor.log_llm_call(...)` |
| `log_tool_call()` | Log tool usage | `monitor.log_tool_call(...)` |
| `log_rag_retrieval()` | Log RAG retrieval | `monitor.log_rag_retrieval(...)` |

## Tracked Metrics

### Automatic (via Callback)
- Chain execution time
- Number of LLM calls
- Number of tool calls
- Agent steps
- Token usage (if available)
- Latency

### Manual
- Custom chain metrics
- RAG retrieval quality
- Tool performance
- Cost tracking
- User satisfaction

## Configuration Options

```python
monitor = LangChainMonitor(
    client=client,
    application_name="my_app",     # App identifier
    track_tokens=True,              # Track token usage
    track_cost=True                 # Track API costs
)
```

## Integration Patterns

### Pattern 1: Global Callback
```python
# One callback for entire app
callback = monitor.create_callback_handler()

# Use everywhere
chain1.run(..., callbacks=[callback])
chain2.run(..., callbacks=[callback])
agent.run(..., callbacks=[callback])
```

### Pattern 2: Per-Component
```python
# Different monitors for components
qa_monitor = LangChainMonitor(client, application_name="qa")
agent_monitor = LangChainMonitor(client, application_name="agent")

qa_callback = qa_monitor.create_callback_handler()
agent_callback = agent_monitor.create_callback_handler()

qa_chain.run(..., callbacks=[qa_callback])
agent.run(..., callbacks=[agent_callback])
```

### Pattern 3: Wrapped Chains
```python
# Wrap for auto-logging
wrapped_chain = wrap_langchain_chain(chain, monitor)

# No callbacks needed
result = wrapped_chain.run(...)
```

## Supported Components

| Component | Support | Notes |
|-----------|---------|-------|
| LLMChain | ✅ | Full support |
| SequentialChain | ✅ | Full support |
| SimpleSequentialChain | ✅ | Full support |
| Agents (all types) | ✅ | Full support |
| Tools | ✅ | Auto-tracked |
| Memory | ✅ | Works with callbacks |
| Retrievers | ✅ | RAG support |
| Custom Chains | ✅ | Use callbacks |

## Best Practices

1. **Register Once**
   ```python
   # ✅ At startup
   monitor.register_application(name="App")
   callback = monitor.create_callback_handler()
   ```

2. **Reuse Callbacks**
   ```python
   # ✅ Create once, reuse many times
   callback = monitor.create_callback_handler()
   for query in queries:
       chain.run(input=query, callbacks=[callback])
   ```

3. **Handle Errors**
   ```python
   try:
       result = chain.run(..., callbacks=[callback])
   except Exception as e:
       print(f"Failed: {e}")
       # Callback still logged partial data
   ```

4. **Track Costs**
   ```python
   # Enable for budget monitoring
   monitor = LangChainMonitor(
       client=client,
       application_name="app",
       track_cost=True
   )
   ```

5. **Use Environment-Specific Apps**
   ```python
   # Development
   dev_monitor = LangChainMonitor(client, application_name="app_dev")

   # Production
   prod_monitor = LangChainMonitor(client, application_name="app_prod")
   ```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Import error | `pip install langchain` |
| Callbacks not working | Use `callbacks=[callback]` (list) |
| Missing tokens | Check LLM provider config |
| High overhead | Set `track_tokens=False` |
| Multiple callbacks | Pass list: `callbacks=[cb1, cb2, wb_callback]` |

## Examples

Full examples in:
- `sdk/examples/langchain_example.py`
- `sdk/guides/LANGCHAIN_INTEGRATION.md`

## Resources

- LangChain Docs: https://python.langchain.com/
- WhiteBoxXAI Docs: https://docs.whiteboxxai.com
- Integration Guide: `sdk/guides/LANGCHAIN_INTEGRATION.md`
