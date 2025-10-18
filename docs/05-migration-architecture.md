# Migration Architecture: Streamlit → Microfrontends + Serverless

**Status:** 📋 Planning Document (Q1-Q2 2025)
**Purpose:** Technical blueprint for migrating from Streamlit monolith to decoupled React architecture
**Target:** Production-ready, independently deployable, highly scalable system

---

## Table of Contents

1. [Current State Analysis](#current-state-analysis)
2. [Target Architecture](#target-architecture)
3. [Migration Strategy](#migration-strategy)
4. [Technology Stack](#technology-stack)
5. [Implementation Roadmap](#implementation-roadmap)
6. [Cost Analysis](#cost-analysis)
7. [Risk Mitigation](#risk-mitigation)

---

## Current State Analysis

### The Streamlit Monolith

**Repository:** `llm_portfolio_assistant/app.py` (5,736 lines)

**Architecture Characteristics:**
- **Coupling:** High - UI, business logic, and data access tightly intertwined
- **Testability:** Low - Difficult to test components independently
- **Deployment:** Monolithic - All-or-nothing releases
- **Scalability:** Limited - Streamlit Cloud free tier, ~100 concurrent users max
- **Development Velocity:** Slow - Changes ripple across codebase

**Identified Pain Points:**
1. ❌ **Brittle Codebase:** Change one feature → breaks unrelated features
2. ❌ **Poor Separation of Concerns:** UI rendering mixed with RAG pipeline logic
3. ❌ **Limited Scalability:** Can't scale components independently
4. ❌ **Deployment Risk:** One bug takes down entire app
5. ❌ **Developer Experience:** 5.7K line file is hard to navigate and maintain

**Current Tech Stack:**
```
Frontend/Backend: Streamlit (Python monolith)
LLM: OpenAI GPT-4
Vector DB: Pinecone (Free tier)
Deployment: Streamlit Cloud
Dependencies: Python venv + requirements.txt
```

---

## Target Architecture

### Design Principles

**1. Decoupling First**
- Each domain (Search, Ask, Stories, Industries) operates independently
- Failure in one domain doesn't cascade to others
- Teams (or future you) can work on features in parallel

**2. Independent Deployability**
- Deploy Search microfrontend without touching Ask MattGPT
- Roll back individual features without full app rollback
- Blue-green deployments per service

**3. Scalability by Domain**
- Auto-scale Ask MattGPT (heavy compute) independently of Search (light compute)
- Pay only for what's used
- Handle 10K+ concurrent users without infrastructure changes

**4. Technology Flexibility**
- Can rewrite Ask MattGPT in different framework without touching Search
- Experiment with new tech in one domain
- Gradual migration path (don't rewrite everything at once)

---

### Microfrontends Architecture

**Pattern:** Module Federation (Runtime Composition)

```
┌─────────────────────────────────────────────────────────┐
│  Shell Application (Host)                               │
│  ├─ Next.js 14 (App Router)                            │
│  ├─ Routing & Navigation                               │
│  ├─ Shared Header/Footer                               │
│  ├─ Authentication (future)                            │
│  └─ Loads microfrontends at RUNTIME                    │
│                                                         │
│  Routes:                                                │
│    / → Load Search MFE                                  │
│    /ask → Load Ask MattGPT MFE                          │
│    /industries/* → Load Industries MFE                  │
│    /story/:id → Load Story Detail MFE                   │
└─────────────────────────────────────────────────────────┘
           │
           ├─► Search MFE (Independent Deploy)
           │   ├─ Next.js app @ search-mfe.mattgpt.com
           │   ├─ Exposes: SearchPage, TableView, CardView
           │   ├─ Calls: lambda-search-* functions
           │   └─ Team: Can deploy without coordinating
           │
           ├─► Ask MattGPT MFE (Independent Deploy)
           │   ├─ Next.js app @ ask-mfe.mattgpt.com
           │   ├─ Exposes: ChatInterface, ConversationHistory
           │   ├─ Calls: lambda-ask-* functions
           │   └─ Team: Can deploy without coordinating
           │
           ├─► Industries MFE (Independent Deploy)
           │   ├─ Next.js app @ industries-mfe.mattgpt.com
           │   ├─ Exposes: IndustryLanding, CategoryBrowse
           │   ├─ Calls: lambda-industries-* functions
           │   └─ Team: Can deploy without coordinating
           │
           └─► Story Detail MFE (Independent Deploy)
               ├─ Next.js app @ stories-mfe.mattgpt.com
               ├─ Exposes: StoryDetail, RelatedStories
               ├─ Calls: lambda-stories-* functions
               └─ Team: Can deploy without coordinating
```

**Module Federation Config Example:**
```javascript
// mattgpt-search-mfe/next.config.js
module.exports = {
  webpack: (config) => {
    config.plugins.push(
      new ModuleFederationPlugin({
        name: 'search',
        filename: 'static/chunks/remoteEntry.js',
        exposes: {
          './SearchPage': './src/pages/search',
          './TableView': './src/components/TableView',
          './CardView': './src/components/CardView',
        },
        shared: {
          react: { singleton: true, requiredVersion: '^18.0.0' },
          'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
        },
      })
    );
    return config;
  },
};

// mattgpt-shell/pages/search.tsx
import dynamic from 'next/dynamic';

const SearchPage = dynamic(() => import('search/SearchPage'), {
  ssr: false,
  loading: () => <div>Loading search...</div>,
});

export default function Search() {
  return <SearchPage />;
}
```

---

### Serverless Backend Architecture (AWS Lambda + FaaS)

**Pattern:** Function-as-a-Service with Domain Isolation

```
AWS API Gateway (REST API)
├─ /api/search/*
│   ├─ POST /semantic → lambda-search-semantic
│   ├─ POST /keyword → lambda-search-keyword
│   └─ POST /hybrid → lambda-search-hybrid
│
├─ /api/ask/*
│   ├─ POST /generate → lambda-ask-generate
│   ├─ POST /rerank → lambda-ask-rerank
│   └─ GET /history → lambda-ask-history
│
├─ /api/stories/*
│   ├─ GET /:id → lambda-stories-get
│   ├─ GET / → lambda-stories-list
│   └─ GET /:id/related → lambda-stories-related
│
└─ /api/industries/*
    ├─ GET /:industry → lambda-industries-get
    └─ GET /:industry/categories → lambda-categories-list
```

**Lambda Function Example:**
```python
# lambda-search-semantic/handler.py

import json
from pinecone_service import search_pinecone  # From Lambda Layer
from openai_service import embed_query       # From Lambda Layer

def lambda_handler(event, context):
    """
    Single Responsibility: Semantic search via Pinecone

    Input: {"query": "banking transformation", "top_k": 20}
    Output: {"results": [...], "count": 15}
    """
    try:
        body = json.loads(event.get('body', '{}'))
        query = body.get('query', '')
        top_k = body.get('top_k', 20)

        # 1. Embed query using OpenAI
        embedding = embed_query(query)

        # 2. Search Pinecone
        results = search_pinecone(embedding, top_k=top_k)

        # 3. Return structured response
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            'body': json.dumps({
                'results': results,
                'count': len(results),
                'query': query
            })
        }

    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }

# Deploy independently:
# sam deploy --stack-name mattgpt-search-semantic
```

**Shared Lambda Layer (Reusable Code):**
```
lambda-layer-common/
└─ python/
    ├─ pinecone_service.py    # Pinecone client wrapper
    ├─ openai_service.py      # OpenAI client wrapper
    ├─ storage_service.py     # DynamoDB/S3 utilities
    └─ requirements.txt       # pinecone-client, openai, boto3
```

---

### Communication Patterns

#### Synchronous (Request/Response)
```
User → Shell → Search MFE → API Gateway → lambda-search-semantic → Pinecone → Response
                                                                 ↓
                                                           DynamoDB (cache)
```

#### Asynchronous (Event-Driven)
```
Lambda (Search) → EventBridge → Lambda (Analytics)
                             → Lambda (ML Training)
                             → Lambda (Notifications)

Example Event:
{
  "event": "search.executed",
  "timestamp": "2025-01-15T10:30:00Z",
  "data": {
    "query": "banking transformation",
    "results_count": 15,
    "user_id": "anonymous",
    "filters": {"industry": "financial_services"}
  }
}
```

**Benefits:**
- Search Lambda doesn't care who consumes events
- Analytics runs asynchronously (doesn't slow down search)
- Easy to add new event consumers later

---

### Data Architecture

**Decoupled Data Stores (Domain Ownership)**

```
Stories Domain:
├─ DynamoDB: stories-table
│   └─ Stores STAR stories, metadata, 5P tags
└─ Pinecone: mattgpt-stories-index
    └─ Stores story embeddings for semantic search

Search Cache Domain:
└─ DynamoDB: search-cache-table
    └─ Stores frequent query results (TTL: 1 hour)

Ask MattGPT Domain:
├─ DynamoDB: conversations-table
│   └─ Stores chat history per session
└─ S3: conversation-exports
    └─ Archived conversations for ML training

Analytics Domain:
├─ DynamoDB: events-table
│   └─ Stores user interaction events
└─ S3: analytics-data
    └─ Aggregated metrics for Athena queries
```

**No Shared Database:** Each domain owns its data. Cross-domain queries happen via APIs.

---

## Migration Strategy

### Phase 0: Preparation (Weeks 1-4)

**Goal:** Extract backend logic without touching Streamlit UI

#### Week 1-2: Set Up Infrastructure

1. **Create AWS Account & Configure**
   - Set up AWS SAM CLI
   - Create Lambda execution role
   - Configure Pinecone API keys in AWS Secrets Manager

2. **Set Up Local Development**
   - Install AWS SAM CLI: `brew install aws-sam-cli`
   - Create basic Lambda template
   - Test local invoke: `sam local invoke`

#### Week 3-4: Extract First Lambda (Proof of Concept)

**Start with simplest domain:** Story retrieval

```python
# lambda-stories-get/handler.py (NEW)

import json
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('stories-table')

def lambda_handler(event, context):
    story_id = event['pathParameters']['id']

    response = table.get_item(Key={'id': story_id})
    story = response.get('Item')

    if not story:
        return {'statusCode': 404, 'body': json.dumps({'error': 'Story not found'})}

    return {
        'statusCode': 200,
        'body': json.dumps(story)
    }
```

**Deploy & Test:**
```bash
sam build
sam deploy --guided
curl https://api.mattgpt.com/stories/123  # Test it works
```

**Success Criteria:**
- ✅ Lambda deployed and invocable
- ✅ Returns story JSON
- ✅ < 500ms response time
- ✅ Doesn't affect Streamlit app

---

### Phase 1: Backend Migration (Weeks 5-12)

**Goal:** Migrate all business logic to AWS Lambdas (keep Streamlit as UI)

#### Week 5-6: Search Domain

Extract search logic from `app.py` → Lambda functions:

```
Lambdas to Create:
├─ lambda-search-semantic (Pinecone semantic search)
├─ lambda-search-keyword (Text matching)
└─ lambda-search-hybrid (Combine semantic + keyword)
```

**Migration Pattern:**
1. Copy logic from `app.py`
2. Create Lambda handler
3. Test independently
4. Update Streamlit to call Lambda (instead of local function)
5. Verify behavior unchanged

#### Week 7-8: Ask MattGPT Domain

```
Lambdas to Create:
├─ lambda-ask-generate (GPT-4 response generation)
├─ lambda-ask-rerank (Rerank search results)
└─ lambda-ask-history (Conversation persistence)
```

#### Week 9-10: Stories Domain

```
Lambdas to Create:
├─ lambda-stories-get (Individual story retrieval)
├─ lambda-stories-list (Browse/filter stories)
└─ lambda-stories-related (Find related stories)
```

#### Week 11-12: Industries Domain

```
Lambdas to Create:
├─ lambda-industries-get (Industry stats)
└─ lambda-categories-list (Category breakdown)
```

**Milestone: Backend Complete**
- ✅ All business logic in Lambdas
- ✅ Streamlit app calls Lambdas via API
- ✅ No breaking changes to user experience

---

### Phase 2: Frontend Migration (Weeks 13-20)

**Goal:** Build React microfrontends (keep Lambdas, replace Streamlit)

#### Week 13-14: Shell Application

**Create container app:**
```bash
npx create-next-app@latest mattgpt-shell
cd mattgpt-shell
npm install @module-federation/nextjs-mf
```

**Set up routing:**
```typescript
// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Header />
        <main>{children}</main>
        <Footer />
      </body>
    </html>
  );
}

// app/page.tsx → Loads Search MFE
// app/ask/page.tsx → Loads Ask MFE
// app/industries/[slug]/page.tsx → Loads Industries MFE
// app/story/[id]/page.tsx → Loads Story Detail MFE
```

**Deploy to Vercel:**
```bash
vercel deploy
# → https://mattgpt.vercel.app
```

#### Week 15-16: Story Detail MFE (Simplest First)

**Why start here:** Single page, no complex state, straightforward API call

```bash
npx create-next-app@latest mattgpt-stories-mfe
```

**Implement:**
```typescript
// src/pages/[id].tsx
import { useStory } from '../hooks/useStory';

export default function StoryDetail({ id }) {
  const { data: story, isLoading } = useStory(id);

  if (isLoading) return <LoadingSkeleton />;

  return (
    <div>
      <h1>{story.title}</h1>
      <STARBreakdown story={story} />
      <MetricsPanel metrics={story.results} />
      <RelatedStories storyId={id} />
    </div>
  );
}

// src/hooks/useStory.ts
import { useQuery } from '@tanstack/react-query';

export function useStory(id: string) {
  return useQuery({
    queryKey: ['story', id],
    queryFn: () => fetch(`/api/stories/${id}`).then(r => r.json()),
  });
}
```

**Module Federation:**
```javascript
// next.config.js
module.exports = {
  webpack: (config) => {
    config.plugins.push(
      new ModuleFederationPlugin({
        name: 'stories',
        filename: 'static/chunks/remoteEntry.js',
        exposes: {
          './StoryDetail': './src/pages/[id]',
        },
        shared: ['react', 'react-dom'],
      })
    );
    return config;
  },
};
```

**Deploy:**
```bash
vercel deploy
# → https://stories-mfe.vercel.app
```

**Integration Test:**
```typescript
// mattgpt-shell/app/story/[id]/page.tsx
import dynamic from 'next/dynamic';

const StoryDetail = dynamic(() => import('stories/StoryDetail'), {
  ssr: false,
});

export default function StoryPage({ params }) {
  return <StoryDetail id={params.id} />;
}
```

**Success Criteria:**
- ✅ Story Detail MFE loads in Shell
- ✅ Fetches data from lambda-stories-get
- ✅ Can deploy independently
- ✅ Matches Streamlit UX (feature parity)

#### Week 17-18: Search MFE

**More complex:** Multiple views (table/card/timeline), filtering, sorting

```
src/
├─ pages/
│   └─ index.tsx           # Main search page
├─ components/
│   ├─ SearchBar.tsx
│   ├─ TableView.tsx
│   ├─ CardView.tsx
│   ├─ TimelineView.tsx
│   └─ FilterPanel.tsx
├─ hooks/
│   ├─ useSearch.ts
│   └─ useFilters.ts
└─ api/
    └─ searchClient.ts
```

#### Week 19-20: Ask MattGPT MFE

**Most complex:** Real-time chat, conversation state, source citations

```
src/
├─ pages/
│   └─ index.tsx           # Chat interface
├─ components/
│   ├─ ChatInput.tsx
│   ├─ MessageList.tsx
│   ├─ Message.tsx
│   └─ SourceChips.tsx
├─ hooks/
│   ├─ useChat.ts          # WebSocket or polling
│   └─ useConversation.ts
└─ state/
    └─ chatStore.ts        # Zustand for local state
```

---

### Phase 3: Production Hardening (Weeks 21-24)

**Goal:** Make it bulletproof

#### Week 21: Performance Optimization

**Metrics to Achieve:**
- Lighthouse Score: > 90
- Time to Interactive: < 3s
- First Contentful Paint: < 1.5s

**Optimizations:**
- Code splitting (React.lazy)
- Image optimization (next/image)
- Lambda cold start reduction (keep-warm)
- API response caching (CloudFront)

#### Week 22: Error Handling & Monitoring

**Add:**
- Sentry for frontend error tracking
- CloudWatch for Lambda logs
- Custom metrics dashboard
- Alert rules (error rate > 5%)

#### Week 23: Testing

**Coverage Target: > 80%**
```bash
# Unit tests
npm test

# E2E tests (Playwright)
npm run e2e

# Load testing (Artillery)
artillery run load-test.yml
```

#### Week 24: Documentation & Handoff

**Create:**
- Deployment runbook
- Incident response guide
- Architecture decision records (ADRs)
- Contributing guidelines

---

## Technology Stack

### Frontend

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Framework | Next.js 14 (App Router) | React framework with SSR/SSG |
| Language | TypeScript | Type safety and developer experience |
| State Management | Zustand | Lightweight local state |
| Server State | TanStack Query | API caching and sync |
| Styling | Tailwind CSS + Shadcn/ui | Utility-first CSS + components |
| Animations | Framer Motion | Smooth transitions |
| Module Federation | @module-federation/nextjs-mf | Runtime composition |

### Backend

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Runtime | AWS Lambda (Python 3.11) | Serverless compute |
| API Gateway | AWS API Gateway (REST) | API routing and throttling |
| IaC | AWS SAM | Infrastructure as code |
| LLM | OpenAI GPT-4 | Natural language processing |
| Vector DB | Pinecone | Semantic search |
| Data Store | DynamoDB | NoSQL database |
| Storage | S3 | Object storage |
| Events | EventBridge | Asynchronous messaging |

### DevOps

| Layer | Technology | Purpose |
|-------|-----------|---------|
| CI/CD | GitHub Actions | Automated testing and deployment |
| Hosting (Frontend) | Vercel | Next.js edge deployment |
| Hosting (Backend) | AWS Lambda | Serverless execution |
| Monitoring | CloudWatch + Sentry | Logs, metrics, errors |
| Secrets | AWS Secrets Manager | API key management |

---

## Implementation Roadmap

### Summary Timeline

```
Phase 0: Preparation               Weeks 1-4   (Backend extraction setup)
Phase 1: Backend Migration         Weeks 5-12  (All logic → Lambdas)
Phase 2: Frontend Migration        Weeks 13-20 (React MFEs)
Phase 3: Production Hardening      Weeks 21-24 (Polish & launch)

Total: 24 weeks (6 months)
```

### Milestones

| Milestone | Date | Deliverable |
|-----------|------|-------------|
| M1: First Lambda Deployed | Week 4 | lambda-stories-get working |
| M2: Backend Complete | Week 12 | All Lambdas deployed, Streamlit uses them |
| M3: First MFE Deployed | Week 16 | Story Detail MFE in production |
| M4: All MFEs Deployed | Week 20 | Full React app replacing Streamlit |
| M5: Production Launch | Week 24 | Streamlit decommissioned, React live |

---

## Cost Analysis

### Current Cost (Streamlit)

```
Streamlit Cloud: $0/month (free tier)
Pinecone: $0/month (free tier, 1 index)
OpenAI: ~$20-50/month (based on usage)

Total: $20-50/month
```

### Target Cost (Microfrontends + Serverless)

```
Frontend Hosting (Vercel):
- Free tier: $0/month
- Pro (if needed): $20/month

Backend (AWS Lambda):
- Free tier: 1M requests/month FREE
- Beyond: $0.20 per 1M requests
- Estimated (10K users): ~$5/month

API Gateway:
- $3.50 per million requests
- Estimated: ~$3/month

DynamoDB:
- On-demand pricing
- Estimated: ~$10/month

S3 Storage:
- $0.023 per GB
- Estimated: ~$2/month

Pinecone:
- Starter: $70/month (when scaling beyond free)
- OR: Replace with Postgres + pgvector: ~$15/month

OpenAI:
- Same as current: ~$20-50/month

CloudWatch:
- Logs + metrics: ~$5/month

Total (Conservative): $50-100/month
Total (with Pinecone): $120-150/month
```

**ROI Analysis:**
- Cost increase: ~$100/month
- Performance gain: 10x (faster, more reliable)
- Scalability: 100 users → 10K+ users
- Development velocity: 3x (independent deployments)

---

## Risk Mitigation

### Technical Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Lambda cold starts | High latency | Keep-warm strategy, provisioned concurrency |
| Module Federation complexity | Integration issues | Start with one MFE, test thoroughly |
| Cost overrun | Budget exceeded | Set CloudWatch alarms, implement caching |
| Data migration | Data loss | Blue-green deployment, backups |
| Pinecone limitations (free tier) | Feature constraints | Plan Pinecone upgrade OR migrate to pgvector |

### Strategic Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Over-engineering | Wasted time | Incremental migration, validate each phase |
| Scope creep | Delayed launch | Stick to roadmap, defer nice-to-haves |
| Learning curve | Slow progress | Invest in learning upfront, build POCs |
| Loss of momentum | Never complete | Set hard deadlines, ship MVPs |

---

## Success Criteria

**Technical:**
- ✅ All Lambdas deployed and functional
- ✅ All MFEs deployed independently
- ✅ 95%+ test coverage
- ✅ < 500ms p95 latency for API calls
- ✅ Lighthouse score > 90

**Business:**
- ✅ Feature parity with Streamlit
- ✅ No user-facing bugs in production
- ✅ Can handle 10K concurrent users
- ✅ Monthly cost < $150

**Developer Experience:**
- ✅ Can deploy individual MFE in < 5 minutes
- ✅ Can add new feature without touching other domains
- ✅ CI/CD pipeline runs in < 10 minutes
- ✅ Documentation complete and up-to-date

---

## Conclusion

This migration strategy balances **ambition with pragmatism**:

- **Demonstrates Enterprise Architecture:** Shows hiring managers you can design distributed systems
- **Incremental Execution:** Doesn't require Big Bang rewrite, can ship in phases
- **Risk Mitigation:** Keeps Streamlit running until React is ready
- **Future-Proof:** Architecture scales to real production needs

**Recommended Approach:**
1. Ship polished Streamlit NOW (1-2 weeks) for job search
2. Document this architecture (shows design thinking)
3. Build one MFE + one Lambda (proves technical skill)
4. Use in interviews to demonstrate strategic thinking
5. Complete full migration when you have a job offer and time

**Remember:** This document is as valuable as the code. It shows you can think architecturally, not just code tactically.

---

**Related Documentation:**
- [Product Vision](/mattgpt-design-spec/docs/01-product-vision) - Strategic positioning and user personas
- [Technical Architecture](/mattgpt-design-spec/docs/02-technical-architecture) - Current phase roadmap
- [UX Design Process](/mattgpt-design-spec/docs/03-ux-design-process) - Wireframes and interaction design
- [Building MattGPT](/mattgpt-design-spec/docs/04-building-mattgpt) - Development journey

---

*Last Updated: October 2024*
*Status: Planning / Reference Architecture*
*Author: Matt Pugmire with Claude Code assistance*
