# opeai-agent-sdk-demo

# OpenAI Agent SDK Demo

This repository demonstrates various use cases of the OpenAI Agent SDK, including web search, RAG (Retrieval-Augmented Generation) with file search, hands-off automation, and guardrails implementation.

## 🚀 Features

- **Web Search Tool**: Uses OpenAI Agent SDK to fetch real-time information from the web.
- **RAG with File Search Tool**: Implements document-based retrieval to enhance responses with file data.
- **Hands-Off Automation**: Automates decision-making and actions based on queries.
- **Guardrails**: Implements constraints and safety checks to ensure responsible AI behavior.

---

## 📂 Setup & Installation

### Prerequisites
- Python 3.11+
- OpenAI SDK
- Required dependencies (see `requirements.txt`)

### Installation Steps

1. Clone the repository:
    ```bash
    git clone https://github.com/your-username/openai-agent-demo.git
    cd openai-agent-demo
    ```
2. Install dependencies:
    ```bash
    pip install -r requirements.txt
    ```
3. Set up environment variables:
    ```bash
    export OPENAI_API_KEY="your_openai_api_key"
    ```

---

## 🛠️ Use Cases

### 1️⃣ Web Search Tool

Uses OpenAI's agent SDK to fetch real-time information.

```python
from openai import OpenAI

stock_agent = Agent(
    name="StockAnalysisAgent",
    instructions="Fetch real-time stock data from Yahoo Finance, analyze trends, and provide insights.",
    tools=[WebSearchTool()]  # Enable web search capability
)


search_query = f"{ticker} stock price site:finance.yahoo.com"
    
prompt = f"Search for '{search_query}' and extract the latest closing price, 50-day SMA, 200-day SMA, and volatility."

# Run the agent with prompt
response = await Runner.run(stock_agent, prompt)

```

### 2️⃣ RAG with File Search Tool

Enhances responses by retrieving relevant data from uploaded files.
We need to 
1. create a Vector DB on OpenAI platform https://platform.openai.com/storage/vector_store. Each Vector DB will have a unique ID to be used by the agent
2. Upload file to the Vector DB

```python
rag_agent = Agent(
    name="RAGAgent",
    instructions="Search for information in Vector DB.",
    tools=[FileSearchTool(vector_store_ids=["vs_67d8446c43ec8191a38f8d905906ba01"])]   
)
print(f" Agent created: {rag_agent.name}")
response = await Runner.run(rag_agent, prompt)
print(response.final_output)
```

### 3️⃣ Hands-Off Automation

Automates decisions and workflows. Here in the workflow 

- **NewsFetcherAgent** → Uses WebSearchTool to retrieve AI news.
- **SummarizerAgent** → Uses FunctionTool to summarize news.
- **InsightsAgent** → Uses FunctionTool to generate AI trends.
- **TriageAgent** → Routes queries between the Summarizer and Insights Generator based on relevance.


```python
triage_agent = Agent(
    name="TriageAgent",
    instructions="Route the query to either the Summarizer or Insights Agent based on AI relevance.",
    handoffs=[news_fetcher_agent, summarizer_agent, insights_agent],
    input_guardrails=[ ai_guardrail],
)
```

### 4️⃣ Guardrails

Implements safety and ethical constraints.

```python
@input_guardrail
async def ai_guardrail(ctx: RunContextWrapper[None], agent: Agent, input_data: str | list[TResponseInputItem]
) -> GuardrailFunctionOutput:
    result = await Runner.run(guardrail_agent, input_data, context=ctx.context)
    final_output = result.final_output_as(AIQueryOutput)
    
    # Debugging prints
    print(f"[Guardrail] User Input: {input_data}")
    print(f"[Guardrail] Validation Result: {final_output.is_ai_related} - {final_output.reasoning}")

    # Ensure correct logic
    return GuardrailFunctionOutput(
        output_info=final_output,
        tripwire_triggered=not final_output.is_ai_related,  # Only trigger if it's NOT AI-related
    )
```

---
