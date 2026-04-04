# Archetype: AI / ML Application

## Default Stack Recommendation

| Layer | Default | Alternative | When to Switch |
|-------|---------|-------------|----------------|
| Framework | Next.js 15 (App Router) | Python (FastAPI) | Backend-only AI service, no UI |
| Language | TypeScript (strict) | Python | Python for ML-heavy, data pipelines, or Jupyter-centric workflows |
| Styling | Tailwind CSS v4 | — | Always Tailwind for UI |
| Components | shadcn/ui | — | Always for React UIs |
| AI SDK | Vercel AI SDK | LangChain.js, Anthropic SDK direct | LangChain for complex chains, direct SDK for simple use |
| LLM Provider | Anthropic (Claude) | OpenAI (GPT), Google (Gemini) | Based on use case: Claude for reasoning, GPT for broad ecosystem, Gemini for multimodal |
| Vector Database | Pinecone | Weaviate, Chroma, Supabase pgvector | Chroma for local/simple, pgvector if already using Supabase, Weaviate for self-hosted |
| Database | Supabase (Postgres + pgvector) | Neon + Pinecone | Supabase if you want vector search + relational in one |
| ORM | Drizzle | Prisma | Prisma for DX, Drizzle for performance |
| Auth | Clerk | NextAuth | Clerk for fast setup, NextAuth for budget |
| File Processing | Unstructured.io | LlamaParse, custom | LlamaParse for PDFs, custom for simple text |
| Hosting | Vercel (frontend) + Railway (workers) | AWS (Lambda + SQS) | AWS for enterprise, high-volume processing |
| Package Manager | pnpm | npm | — |

## Default Directory Structure

```
src/
  app/
    (app)/
      chat/page.tsx                # Main chat interface
      chat/[id]/page.tsx           # Chat history / conversation
      documents/page.tsx           # Document management (for RAG)
      documents/upload/page.tsx    # Upload interface
      settings/page.tsx            # API keys, model preferences
      layout.tsx                   # App layout with sidebar (conversations list)
    (marketing)/
      page.tsx                     # Landing page
      pricing/page.tsx             # Pricing / usage tiers
      layout.tsx
    api/
      chat/route.ts                # Streaming chat endpoint
      documents/
        upload/route.ts            # Document upload + processing
        search/route.ts            # Vector similarity search
      webhooks/
        stripe/route.ts            # Billing webhook
    layout.tsx
    globals.css
  components/
    ui/                            # shadcn/ui primitives
    chat/
      ChatInterface.tsx            # Main chat area
      MessageBubble.tsx            # Individual message (user/assistant)
      StreamingMessage.tsx         # Animated streaming response
      ChatInput.tsx                # Input with send button, file attach
      ConversationList.tsx         # Sidebar list of past conversations
    documents/
      DocumentUploader.tsx         # Drag-and-drop file upload
      DocumentList.tsx             # List of uploaded documents
      SourceCitation.tsx           # Shows which documents were referenced
    shared/
      ModelSelector.tsx            # Dropdown to pick AI model
      TokenCounter.tsx             # Shows token usage
      CopyButton.tsx               # Copy response to clipboard
  lib/
    ai/
      client.ts                    # AI SDK client setup
      prompts.ts                   # System prompts, prompt templates
      models.ts                    # Model configuration (which models, params)
    vector/
      client.ts                    # Vector DB client (Pinecone/pgvector)
      embeddings.ts                # Generate embeddings from text
      search.ts                    # Similarity search functions
    documents/
      parser.ts                    # Extract text from uploaded files
      chunker.ts                   # Split text into chunks for embedding
      processor.ts                 # Full pipeline: upload → parse → chunk → embed → store
    db/
      schema.ts                    # Drizzle schema
      index.ts                     # Database client
    utils.ts
  types/
    index.ts
```

## Common Patterns

### Chat with Streaming (Vercel AI SDK)

```typescript
// api/chat/route.ts
import { anthropic } from '@ai-sdk/anthropic'
import { streamText } from 'ai'

export async function POST(req: Request) {
  const { messages, conversationId } = await req.json()

  // Optional: retrieve relevant context from vector DB
  const context = await vectorSearch(messages[messages.length - 1].content)

  const result = streamText({
    model: anthropic('claude-sonnet-4-5-20250514'),
    system: buildSystemPrompt(context),
    messages,
  })

  return result.toDataStreamResponse()
}

// Client component
'use client'
import { useChat } from 'ai/react'

function ChatInterface() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat()
  // Render messages + input form
}
```

### RAG (Retrieval Augmented Generation)

```
User uploads document
  → Parse text (PDF, DOCX, TXT)
  → Split into chunks (500-1000 tokens each, with overlap)
  → Generate embeddings for each chunk (OpenAI/Cohere embeddings API)
  → Store chunks + embeddings in vector database

User asks a question
  → Generate embedding of the question
  → Search vector DB for similar chunks (top 5-10)
  → Inject retrieved chunks into system prompt as context
  → LLM answers using the provided context
  → Show source citations to user
```

### Document Processing Pipeline

```typescript
// lib/documents/processor.ts
async function processDocument(file: File, userId: string) {
  // 1. Parse: extract raw text
  const text = await parseDocument(file)

  // 2. Chunk: split into overlapping segments
  const chunks = splitIntoChunks(text, {
    chunkSize: 800,     // tokens per chunk
    overlap: 100,       // overlap between chunks
  })

  // 3. Embed: generate vectors for each chunk
  const embeddings = await generateEmbeddings(chunks)

  // 4. Store: save to vector DB with metadata
  await vectorStore.upsert(chunks.map((chunk, i) => ({
    id: `${file.name}-${i}`,
    values: embeddings[i],
    metadata: {
      text: chunk,
      source: file.name,
      userId,
      chunkIndex: i,
    },
  })))
}
```

### Token Usage Tracking

```sql
usage_logs
  id            UUID PRIMARY KEY
  user_id       UUID REFERENCES users(id)
  model         TEXT NOT NULL           -- claude-sonnet-4-5-20250514, gpt-4o
  input_tokens  INTEGER NOT NULL
  output_tokens INTEGER NOT NULL
  cost_cents    INTEGER NOT NULL        -- cost in cents for billing
  conversation_id UUID
  created_at    TIMESTAMPTZ DEFAULT now()

-- Index for billing queries
CREATE INDEX idx_usage_user_month ON usage_logs(user_id, created_at);
```

### Prompt Management

```typescript
// lib/ai/prompts.ts
export const SYSTEM_PROMPTS = {
  default: `You are a helpful assistant. Answer questions based on the provided context.
If the context doesn't contain relevant information, say so honestly.`,

  rag: (context: string) => `You are a helpful assistant with access to the user's documents.
Answer questions based ONLY on the following context. If the answer is not in the context, say
"I couldn't find that in your documents."

Context:
${context}`,

  codeReview: `You are a senior code reviewer. Review the provided code for:
bugs, security issues, performance, readability. Be direct and specific.`,
}
```

## Build Order

1. **Scaffolding**: Next.js, Tailwind, shadcn/ui, install AI SDK + vector DB client.
2. **AI integration**: Basic chat endpoint with streaming, hardcoded system prompt. Test with curl.
3. **Chat UI**: Chat interface with message bubbles, streaming display, input box.
4. **Conversation history**: Save/load conversations in database. Sidebar with conversation list.
5. **Document upload**: Upload endpoint, file parsing (text extraction from PDF/DOCX/TXT).
6. **RAG pipeline**: Chunking, embedding generation, vector storage, similarity search.
7. **Context injection**: Wire vector search into chat — retrieve relevant chunks, inject into prompt.
8. **Source citations**: Show which documents/chunks were used to generate the answer.
9. **Auth**: Clerk setup, protected routes, per-user document isolation.
10. **Usage tracking**: Log token usage per request, show usage dashboard.
11. **Billing** (if applicable): Usage-based pricing with Stripe metered billing.
12. **Landing page**: Hero, demo, pricing, features.
13. **Polish**: Loading states, error handling, mobile responsive, rate limiting.
14. **Deploy**: Vercel (frontend) + Railway (background processing). Environment variables, monitoring.

## Common Pitfalls

- **Don't send entire documents as context.** Chunk and retrieve only relevant sections. LLM context windows are large but not infinite, and cost scales with input tokens.
- **Don't skip embedding chunk overlap.** Without overlap, you lose context at chunk boundaries. Use 50-200 token overlap.
- **Don't ignore token costs.** Track usage per user. A single RAG query with large context can cost $0.10+. Add rate limiting and usage caps.
- **Don't hardcode prompts.** Store system prompts as configurable strings. You will iterate on them constantly.
- **Don't stream without error handling.** Streams can fail mid-response. Handle partial responses gracefully on the client.
- **Don't forget to handle long conversations.** As conversation grows, truncate or summarize old messages to stay within token limits.
- **Don't store embeddings in your relational database** unless using pgvector. Use a dedicated vector DB for production scale.

## Skills for Build Phase

| Skill | When |
|-------|------|
| `/frontend-design` | Chat interface, document management UI |
| `/shadcn-ui` | Component setup |
| `/deep-research` | Comparing embedding models, vector DBs, chunking strategies |
| `/claude-api` | Setting up Anthropic SDK, streaming, tool use |

## See Also

- `knowledge/building-blocks/api-design-patterns.md` — Streaming endpoints, API design
- `knowledge/building-blocks/database-patterns.md` — pgvector setup, schema design
- `knowledge/building-blocks/auth-patterns.md` — Per-user data isolation
- `knowledge/building-blocks/payments-patterns.md` — Usage-based billing with Stripe
- `knowledge/building-blocks/integrations-patterns.md` — Document upload, storage, email
