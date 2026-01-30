---
title: "AI Tools Shift from General to Vertical: Reflections on Agent Skills, MCP, and RAG in Production"
date: 2026-01-8
categories:
- AI
tags:
  - AI Agent
  - Production AI
  - Agent Skills
  - MCP
  - RAG
  - Vertical AI
  - System Architecture

layout: single
author_profile: true
read_time: true
comments: false
share: true
---

## In the Beginning

Back in 2023, everyone was asking what AI could do. The answers were cool but vague: write essays, generate code, summarize documents. By 2025, the question got more specific: how do I actually deploy AI to cut my market analysis time from 2 hours down to 15 minutes? Not just a change in what people wanted, but a fundamental shift in how AI tools got built.

The gap between demo and production turned out bigger than expected. ChatGPT writes code, sure. But building a reliable coding assistant means handling context windows, managing tool invocations, retrieving relevant docs, maintaining conversation state. That impressive demo hides a lot of engineering complexity.

Past couple years saw a flood of new concepts: RAG, Agent Skills, MCP. Each solves a specific production problem. Looked at individually, they're useful. Put them together, you start seeing a new architectural pattern emerge.

I'll walk through these three patterns and see what they reveal about how AI tools are shifting from general-purpose platforms to vertical applications. From "AI can do X" to "here's a system that reliably does X at production scale."

## Part 1: Agent Skills - Modular Capabilities

### The Problem

Here's the current state: every AI application implements tools independently. An app using OpenAI's API defines a `read_file()` function. Another using Anthropic's API implements the same thing with different format requirements. A third targeting Google's API does it all over again. Fragmented ecosystem, duplicated work everywhere.

The costs compound. File operation logic needs updating—better handling for large files or binary data—each application patches independently. New capabilities emerge like streaming reads, each implementation evolves separately. The industry keeps reinventing the same wheels.

### What Skills Are

Agent Skills are reusable capability modules that AI agents can discover and invoke on their own. File-based format, not code. A skill is just a folder with instructions for the AI, execution scripts, and resources.

Structure looks like this:

```
market-analysis-skill/
  ├── skill.md          # AI-readable instructions
  ├── scripts/
  │   ├── calculate_regime.py
  │   └── fetch_data.py
  └── resources/
      └── thresholds.json
```

The skill.md file contains structured information:

```markdown
name: calculate_market_regime
description: Calculates Fear/Greed regime using network density and market breadth
when_to_use: User asks about market state, regime classification, or risk assessment

tools:
  - fetch_network_data: Retrieves blockchain network metrics
  - calculate_density: Computes network density from transaction graph
  - classify_regime: Applies K-Means clustering to classify regime

implementation: scripts/calculate_regime.py
```

Key thing here is autonomy. Skills give the AI both tools and a decision framework. The AI figures out when and how to use them based on what the user wants. Different from preset workflows where you hardcode the execution sequence.

### Skills vs Function Calling

Comparison on key dimensions:

| Dimension | Function Calling | Agent Skills |
|-----------|-----------------|--------------|
| Granularity | Single function | Complete capability package |
| Distribution | Inline code in application | Standalone folder/npm package |
| Knowledge encoding | Function signature and description | Domain expertise, best practices, error handling patterns |
| Reusability | Application-specific | Cross-application, cross-platform |
| Update mechanism | Requires application code change | Independent skill versioning |
| Ecosystem | Vendor-specific | Open standard (agentskills.io) |

The fundamental difference is abstraction level. Function calling provides primitive operations. Skills provide domain-specific capabilities with embedded expertise.

Here's a concrete example. A `query_database` function accepts SQL strings. A database analysis skill knows when to query, how to construct optimal queries for different scenarios, how to handle connection failures, and how to interpret results in domain context. The function is a tool. The skill is expertise packaged with tools.

```python
# Function calling approach
tools = [
    {"name": "query_database", "parameters": {"sql": "string"}}
]
# AI must figure out: What to query? How to construct SQL? Handle errors?

# Skill approach
# skill.md contains:
# - Query patterns for common scenarios
# - Error handling strategies
# - Result interpretation guidelines
# AI invokes skill with high-level intent, skill handles details
```

### Skills vs Hard-coded Workflows

Common misconception: treating skills as preset execution sequences. The distinction matters.

Hard-coded workflow:

```python
def create_blog_post(title, category):
    template = read_template()
    content = generate_content(title)
    metadata = create_metadata(category)
    save_file(f"{title}.md", template.format(content, metadata))
```

This is deterministic. Steps run in fixed order no matter what.

Skill-based approach:

```python
available_skills = ["read_file", "create_file", "list_directory"]

# AI reasoning process (not explicit code):
# 1. User wants new blog post
# 2. Need to understand format → list_directory to find examples
# 3. Examine example → read_file to learn structure
# 4. Generate content following learned format
# 5. create_file with proper structure
```

The execution sequence emerges from AI reasoning, not preset logic. Different contexts produce different sequences. User provides explicit format requirements? Steps 2-3 get skipped. Format ambiguous? AI might read multiple examples.

This flexibility is the value. Skills provide capabilities, AI composes them based on context.

### Ecosystem Status

The Agent Skills spec lives at agentskills.io as an open standard. Anthropic started it, but the format is vendor-neutral. OpenAI adopted compatible format for Codex CLI. Standardization enables cross-platform skill sharing.

SkillsMP marketplace (skillsmp.com) aggregates skills from GitHub repos. As of January 2026, it indexes 107,805 skills. Distribution is notable. Major tech companies publish skills encoding their domain expertise:

- facebook/react repository: feature-flags skill (242.6k installs) - React team's knowledge about feature flag management
- vercel/next.js repository: cache-components skill (137.4k installs) - Next.js caching patterns
- n8n-io/n8n repository: create-pr skill (171.7k installs) - pull request best practices

Pattern mirrors open source library distribution. Companies that previously open-sourced code now open-source expertise. Shift from "here's our implementation" to "here's how we think about this problem."

Skills distribute via npm, pip, or direct GitHub clones. Installation is file-based. Copy skill folders to `~/.claude/skills/` or `.claude/skills/` for project-specific skills. AI runtime discovers and loads them automatically.

## Part 2: Model Context Protocol - Standardized Tool Layer

### The Fragmentation Problem

AI tool integration in 2024-2025 means platform-specific implementations. Each provider uses different interface formats:

```python
# OpenAI format
openai_tools = [{
    "type": "function",
    "function": {
        "name": "read_file",
        "parameters": {
            "type": "object",
            "properties": {"path": {"type": "string"}},
            "required": ["path"]
        }
    }
}]

# Anthropic format
anthropic_tools = [{
    "name": "read_file",
    "description": "Read file contents",
    "input_schema": {
        "type": "object",
        "properties": {"path": {"type": "string"}},
        "required": ["path"]
    }
}]

# Google Gemini format
gemini_tools = [{
    "function_declarations": [{
        "name": "read_file",
        "parameters": {
            "type": "object",
            "properties": {"path": {"type": "string"}}
        }
    }]
}]
```

Logic is identical. Format differs. Developers maintaining multi-platform apps implement the same file operations three times with different wrappers. Updates need synchronized changes across all implementations.

Impact scales with tool count. App supporting 20 tools across 3 platforms maintains 60 implementations. Tool logic update touches 60 locations. Maintenance burden adds up fast.

### MCP Architecture

Model Context Protocol fixes this by adding an abstraction layer between AI and tools. Architecture follows client-server model:

```
┌─────────────────┐
│   AI Client     │  (Claude, GPT, Gemini)
│  - Intent parse │
│  - Tool select  │
│  - Result synth │
└─────────────────┘
        ↓
   MCP Protocol (JSON-RPC over stdio/HTTP)
        ↓
┌─────────────────┐
│   MCP Server    │  (Tool implementation)
│  - Tool registry│
│  - Execute logic│
│  - Return result│
└─────────────────┘
```

Roles are separated. AI client handles reasoning: understanding user intent, selecting tools, synthesizing results. MCP server handles execution: maintaining tool definitions, running tool logic, returning structured results. Protocol just defines communication format.

Critical point about execution: AI doesn't execute tools. AI decides what to call and with what parameters. Server executes. This separation gives you security isolation, language independence, and independent versioning.

Here's an interaction:

```
AI reasoning: User wants file list
AI decision: Call list_files tool

MCP request (AI → Server):
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "list_files",
    "arguments": {"path": "/path/to/directory"}
  }
}

Server execution:
const files = await fs.readdir(path);

MCP response (Server → AI):
{
  "jsonrpc": "2.0",
  "result": {
    "content": [{
      "type": "text",
      "text": "[\"file1.md\", \"file2.md\"]"
    }]
  }
}

AI synthesis: "The directory contains 2 files: file1.md and file2.md"
```

### Execution Flow

Walk through a complete flow for "List files in _posts directory":

AI receives the query, parses intent, checks available tools via MCP list_tools endpoint, matches it to list_files tool.

AI sends request:

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "tools/call",
  "params": {
    "name": "list_files",
    "arguments": {
      "path": "/Users/user/blog/_posts"
    }
  }
}
```

MCP server gets the request, validates parameters against tool schema, executes:

```typescript
async function handleToolCall(request) {
  if (request.params.name === "list_files") {
    const path = request.params.arguments.path;
    const files = await fs.readdir(path);
    return {
      content: [{
        type: "text",
        text: JSON.stringify(files)
      }]
    };
  }
}
```

Server returns result:

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "result": {
    "content": [{
      "type": "text",
      "text": "[\"2024-11-18-Finance.md\", \"2024-12-03-diamond.md\", ...]"
    }]
  }
}
```

AI gets the result, incorporates it into context, generates response: "Your _posts directory contains 26 markdown files, including recent posts on finance and market analysis."

Server runs in a separate process. This gives you security isolation—server can run in a container with limited permissions. Language independence—server written in whatever language works best, doesn't matter what the AI client uses. Update independence—server updates without restarting the AI client.

### What MCP Solves

MCP addresses systematic pain points:

| Pain Point | Before MCP | With MCP | Impact |
|-----------|-----------|----------|---------|
| Duplicate implementation | Implement per AI platform | Write once, works everywhere | 3x-5x development time reduction |
| Tool updates | Modify all applications, coordinate restarts | Update server independently | Zero downtime updates possible |
| Tool sharing | Copy-paste code, manual adaptation | `npm install mcp-server-name` | Ecosystem effects |
| Security isolation | Tools run in AI process | Server runs in separate process/container | Attack surface reduction |
| Tech stack constraints | Limited by AI application language | Server uses optimal language | Go for performance, Rust for system access, etc. |
| Enterprise integration | Per-application integration | Centralized MCP gateway | N tools × M clients = N implementations (not N×M) |
| Dynamic extension | Modify source code, rebuild | Add configuration entry | No code changes required |

Production impact for enterprise with 100 internal tools: traditional approach needs each AI app to integrate 100 tools separately. Five apps means 500 integration points. MCP approach implements 100 servers once, apps connect via config. Tool logic updates touch one server, not five apps.

Cost example: company building market analysis agent with 20 tools across 3 AI platforms—OpenAI for chat, Anthropic for complex reasoning, Google when multimodal input is needed. Traditional: 60 tool implementations. MCP: 20 servers + 3 client configs. Maintenance drops from 60 to 20 components.

### Technical Implementation

MCP server implementation example (TypeScript):

```typescript
import { Server } from "@modelcontextprotocol/sdk/server";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio";

const server = new Server({
  name: "market-data-server",
  version: "1.0.0"
});

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: "fetch_market_data",
    description: "Retrieve cryptocurrency market data",
    inputSchema: {
      type: "object",
      properties: {
        symbol: { type: "string", description: "Token symbol" },
        timeframe: { type: "string", enum: ["1h", "24h", "7d"] }
      },
      required: ["symbol"]
    }
  }]
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;
  
  if (name === "fetch_market_data") {
    const data = await marketAPI.getData(args.symbol, args.timeframe);
    return {
      content: [{
        type: "text",
        text: JSON.stringify(data)
      }]
    };
  }
  
  throw new Error(`Unknown tool: ${name}`);
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

Client configuration (JSON):

```json
{
  "mcpServers": {
    "market-data": {
      "command": "node",
      "args": ["./market-data-server.js"],
      "env": {
        "API_KEY": "${MARKET_API_KEY}"
      }
    },
    "blockchain-data": {
      "command": "python",
      "args": ["-m", "blockchain_server"]
    }
  }
}
```

Configuration-driven approach enables dynamic server management. Adding new tools needs config entry, not code modification. Infrastructure as configuration, not infrastructure as code.

## Part 3: RAG - External Knowledge Access

### The Knowledge Problem

AI models have finite knowledge boundaries. Training data has cutoff dates. GPT-4 trained through October 2023. Claude 3 through August 2024. My training ends October 2024. Questions about events after that? Can't answer from model knowledge.

Private information is inaccessible. Company internal docs, personal notes, proprietary research—none of that in training data. Model can't access what it wasn't trained on.

Real-time data unavailable. Current stock prices, today's news, latest blockchain transactions aren't encoded in model weights. Static training means no dynamic information access.

User-specific context is absent. Model doesn't know your previous conversations beyond current session, your preferences, your writing style, your domain expertise.

Concrete example: query to an AI trained through October 2024: "Did I write articles about market fear indicators?" Without external knowledge access, response is: "I don't have access to your blog or documents." Model has capability to analyze content, just lacks information access.

Problem isn't capability, it's information. Model is smart enough to analyze market indicators if you give it the data. Constraint is data availability.

### RAG Pipeline

Retrieval-Augmented Generation fixes this with a three-stage pipeline: retrieve relevant documents, augment prompt with those documents, generate response using the augmented context.

First stage: Retrieval. Convert query to vector embedding, search vector database for similar document embeddings, return top-k most similar documents.

```python
# Query vectorization
query = "market fear indicators methodology"
query_embedding = embedding_model.embed(query)
# Result: 1536-dimensional vector

# Similarity search
results = vector_db.similarity_search(
    query_vector=query_embedding,
    top_k=5,
    filter={"category": "Data Mining"}
)
# Returns 5 document chunks with highest cosine similarity
```

Vector database stores pre-computed embeddings for all documents. Similarity search uses cosine similarity between query vector and document vectors. Top-k selection returns most relevant chunks.

Second stage: Augmentation. Combine retrieved documents with original query, construct enhanced prompt.

```python
# Extract document text
context = "\n\n".join([
    f"Document {i+1}:\n{doc.content}"
    for i, doc in enumerate(results)
])

# Construct prompt
prompt = f"""Reference documents:
{context}

User question: {query}

Answer based on the documents provided. Cite specific documents when making claims."""
```

Augmented prompt provides factual grounding. Model generates response based on provided documents rather than potentially hallucinated information.

Third stage: Generation. Pass augmented prompt to language model, generate response.

```python
response = llm.generate(
    prompt=prompt,
    max_tokens=500,
    temperature=0.2  # Lower temperature for factual responses
)
```

Model synthesizes information from documents to answer query. Critical point: response is grounded in retrieved documents, not model's training data.

Flow looks like this:

```
User Query
    ↓
Embedding Model (text → vector)
    ↓
Vector Database
    ├─ Document embeddings (pre-computed)
    └─ Similarity search (cosine)
    ↓
Top-k Document Retrieval
    ↓
Prompt Augmentation (query + documents)
    ↓
Language Model Generation
    ↓
Response (with document citations)
```

Example flow for query "What methodology did I use for market fear analysis?":

Retrieval returns:
- Document 1: "Market Fear Regime Identification - Part 1" (similarity: 0.89)
- Document 2: "Market Fear Regime Identification - Part 2" (similarity: 0.85)
- Document 3: "Market Fear Regime Identification - Part 3" (similarity: 0.82)

Augmentation creates prompt: "Reference documents: [full text of 3 articles]. Question: What methodology did I use for market fear analysis?"

Generation produces: "You used K-Means clustering on four features: 7-day network density, 7-day returns, market breadth, and volatility. The model identified two regimes: Fear (high density 0.89, negative returns, low breadth 22%) and Greed (low density 0.38, positive returns, high breadth 68%). Source: Document 1, Document 3."

### RAG vs Fine-tuning

Two approaches for domain-specific knowledge: Retrieval-Augmented Generation and fine-tuning. Trade-offs differ significantly.

| Dimension | RAG | Fine-tuning |
|-----------|-----|-------------|
| Cost | $0.001-0.01 per query | $100-$10,000 per training run |
| Update latency | Real-time (add to vector DB) | Hours to days (retrain model) |
| Knowledge encoding | External documents | Model weights |
| Explainability | Shows source documents | Black box (cannot trace knowledge source) |
| Accuracy degradation | None (documents unchanged) | Catastrophic forgetting possible |
| Scalability | Scales with document count | Limited by training compute |

Use case guidance:

RAG appropriate when knowledge updates frequently. News monitoring, documentation search, personal knowledge management require real-time updates. Adding new document to vector database takes seconds. Model sees new information immediately.

Fine-tuning appropriate when behavior modification needed. Domain-specific terminology, output format, reasoning style benefit from fine-tuning. Example: Medical diagnosis requires specialized terminology and reasoning patterns. Fine-tuning encodes this into model behavior.

The approaches combine. Fine-tune for domain expertise and output style. Use RAG for factual information. Medical AI example: Fine-tune on medical reasoning patterns and terminology. Use RAG for latest research papers and drug information.

Cost comparison for market analysis agent: RAG approach with 1000 documents, 100 queries per day. Vector DB storage: $5/month. Embedding cost: $0.50/month. Query cost: $0.001 × 100 × 30 = $3/month. Total: $8.50/month.

Fine-tuning approach: Initial training $1000. Retraining monthly for updates $1000/month. No query-time augmentation cost. But knowledge outdated between retraining runs.

For dynamic knowledge domains, RAG provides better cost-performance profile.

### Implementation Considerations

Technical components for production RAG system:

Embedding model selection: text-embedding-ada-002 (OpenAI, 1536 dimensions, $0.0001/1K tokens), text-embedding-3-small (OpenAI, 1536 dimensions, higher quality), sentence-transformers models (open source, local execution).

Vector database options: FAISS (Facebook, local, good for <1M vectors), Pinecone (cloud, scalable), Chroma (open source, developer-friendly), Weaviate (open source, production-grade).

Chunking strategy affects retrieval quality. Documents split into chunks. Chunk size trade-off: small chunks (200-300 tokens) provide precise retrieval but lose context. Large chunks (1000-1500 tokens) preserve context but reduce precision. Optimal range: 500-800 tokens per chunk with 50-100 token overlap between chunks.

Metadata filtering improves relevance. Store metadata with chunks: document date, category, author. Filter before similarity search. Example: "Show only documents from 2024 in Data Mining category." Reduces search space, improves precision.

Reranking improves top-k accuracy. Two-stage retrieval: broad retrieval (top-50 candidates), reranking with cross-encoder (select top-5). Cross-encoder computes query-document similarity more accurately than embedding similarity but is computationally expensive. Use for reranking small candidate set.

Citation tracking maintains traceability. Store chunk IDs with responses. User can verify which documents contributed to answer. Critical for high-stakes domains like legal research or medical diagnosis.

Implementation example for crypto market analysis:

```python
# Index all historical blog posts
from langchain.document_loaders import DirectoryLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.vectorstores import FAISS
from langchain.embeddings import OpenAIEmbeddings

loader = DirectoryLoader('_posts/', glob="**/*.md")
documents = loader.load()

splitter = RecursiveCharacterTextSplitter(
    chunk_size=600,
    chunk_overlap=50
)
chunks = splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
vector_db = FAISS.from_documents(chunks, embeddings)

# Query for similar scenarios
query = "Terra Luna collapse market structure"
results = vector_db.similarity_search(query, k=5)

# Results contain historical analysis of Terra collapse
# Can compare current market to historical scenario
```

Quality factors: embedding model quality affects retrieval accuracy. Higher-quality embeddings provide better semantic similarity. Chunk size affects context vs precision trade-off. Metadata filtering reduces noise. Reranking improves top-k selection.

Implementation complexity is moderate. Vector database setup straightforward. Chunking strategy needs experimentation. Embedding model selection depends on cost-quality trade-off. Production deployment adds monitoring and update pipelines.

## Part 4: Putting Them Together

### Independence

These three address different problems and can work independently.

RAG solves information access. Query private docs, retrieve historical data, access real-time info. Needs vector database, embedding model, retrieval pipeline. No dependency on Skills or MCP. Works with any LLM supporting prompt augmentation.

Skills solve capability reusability. Package domain expertise for cross-platform use, enable community-driven capability sharing. Needs skill folder structure, AI tool discovery mechanism. No dependency on RAG or MCP. Works with any AI supporting tool calling.

MCP solves tool standardization. Unified tool interface across AI platforms, reduce implementation duplication, enable independent tool evolution. JSON-RPC protocol, client-server architecture. No dependency on RAG or Skills. Works as pure protocol layer.

Can adopt incrementally. Implement RAG for knowledge access without changing tool infrastructure. Publish Skills without requiring MCP support. Adopt MCP without mandating Skills format.

### When They Combine

They can complement each other when a use case needs multiple capabilities. Not a requirement, just wandering how they fit together.

Example scenario: analyzing some complex question that needs historical context (RAG retrieves past analysis), domain-specific computation (Skill runs calculations), and real-time data (MCP fetches current metrics). Each piece handles its part. Not because there's a "standard architecture," just because the problem needs all three.

But plenty of systems use just one or two. Documentation search: RAG only. Code assistant: Skills for capabilities, maybe MCP for tool access. API integration: MCP only. Depends on the problem.

## Part 5: The Verticalization Trend

### From Horizontal to Vertical

AI tool evolution shows a clear timeline. 2022-2023 focused on horizontal capability. Question was "what can AI do?" Products demonstrated breadth: write essays, generate code, answer questions, create images. Metric: model performance on benchmarks. User behavior: experimentation, prompt engineering, capability exploration.

2024-2025 shifted to vertical application. Question became "how do I deploy AI for specific task?" Products narrowed focus: GitHub Copilot for code, Perplexity for research, Cursor for IDE integration, NotebookLM for knowledge management. Metric: task completion rate, production reliability, ROI. User behavior: system building, integration work, production deployment.

2026 and beyond represents infrastructure phase. AI transitions from feature to foundation. Focus: cost efficiency, deep integration, multi-agent coordination. Products: agentic workflows, specialized agent teams, domain-specific platforms. User behavior: AI embedded in standard workflows, not separate tool.

Defining characteristic of current phase: shift from demonstration to deployment. From "AI can do X" to "system reliably performs X at scale."

Evidence worth noting:

GitHub Copilot adoption data (GitHub 2024 report): 40% of code in repos with Copilot access is AI-generated. Not experimentation anymore. Production dependency. The tool moved from curiosity to critical infrastructure.

Perplexity vs ChatGPT usage patterns: Perplexity focuses on research with citations. ChatGPT remains general-purpose. Perplexity grew faster in research-heavy user segments. Vertical focus wins in specific domains.

Cursor IDE vs generic code helpers: Cursor integrates deeply with development workflow. Context awareness of entire codebase, project-specific customization. Generic helpers offer broader capability but shallower integration. Cursor captures developer mindshare through depth, not breadth.

NotebookLM for knowledge management: Narrow use case—document analysis, note organization. Deep integration with specific workflow. Outperforms general AI for this task through specialization.

Pattern: success correlates with vertical focus and deep integration, not horizontal capability and broad coverage.

Market validation: vertical AI tools raise larger rounds at higher valuations than horizontal alternatives. Investors recognize production value comes from domain expertise, not general intelligence.

### What next?
We will see more automation and specialization. 
We will seeeee.