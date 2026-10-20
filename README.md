<p align="center">
  <img src="assets/banner.svg" alt="Vendra — multi-tenant AI sales agent: 92.4% answer accuracy, 1.2% hallucination, 95.5% retrieval recall" width="100%"/>
</p>

<p align="center">
  <a href="https://coachsync.ahmedhussain.tech"><img src="https://img.shields.io/website?url=https%3A%2F%2Fcoachsync.ahmedhussain.tech&label=Vendra&up_message=live&down_message=offline&up_color=3DD68C&style=for-the-badge&labelColor=101018" alt="Vendra status"/></a>
  <img src="https://img.shields.io/badge/Agentic%20RAG-101018?style=for-the-badge&logo=langchain&logoColor=3BA4F0" alt="Agentic RAG"/>
  <img src="https://img.shields.io/badge/MCP%20server-101018?style=for-the-badge&logo=anthropic&logoColor=9B8CFF" alt="MCP server"/>
  <img src="https://img.shields.io/badge/OAuth%202.1-101018?style=for-the-badge&logo=auth0&logoColor=3DD68C" alt="OAuth 2.1"/>
</p>

**Vendra gives any business its own AI sales agent.** The agent answers buyers on WhatsApp, in a website chat widget and over Gmail, from that business's own catalog, documents and FAQs. It books meetings, sends Stripe payment links in the chat and hands over to a human at any point. One platform serves many businesses, each fully isolated from the others.

Two things make it more than a chatbot: **every answer is retrieved, graded and checked against its sources before it is sent**, and **other AI assistants can use Vendra as a tool** through its own OAuth-secured MCP server.

## Architecture

<p align="center">
  <img src="assets/architecture.svg" alt="Vendra architecture: channels → Nginx → sales agent → agentic RAG loop → grounding check → retrieval traces; MCP clients → OAuth 2.1 → MCP server" width="100%"/>
</p>

| Part | What it does |
|---|---|
| **Channels** | WhatsApp, a one-script-tag website widget and Gmail all feed the same agent. A new channel is one adapter module; nothing in the agent changes. |
| **Sales agent** | A LangGraph router with specialist agents. It runs on a worker and queue, so a web request never waits on an agent turn. |
| **Agentic RAG loop** | Searches all four knowledge stores at once (catalog, FAQs, documents, business profile), grades every passage, and rewrites the query and searches again when nothing relevant is found. |
| **Grounding check** | Every claim in a draft reply must cite a passage. If it can't, the reply is rewritten from its sources, or replaced with "the team will confirm". |
| **MCP server** | OAuth 2.1 with PKCE, dynamic client registration, a consent page and refresh-token rotation; 9 tools with per-tool scopes, and every call written to an audit log. |
| **Observability** | Every search is stored as a retrieval trace, and the dashboard's *Answer quality* tab shows the sources behind each answer. |

## Measured results

The same 81 questions over two businesses, asked to the same agent twice: once with the original single-shot retrieval, once with the agentic loop and grounding check. Nothing was mocked. Each question went through the real router, specialists, tools and prompts, against a real Postgres + pgvector database.

| Measure | Before (single-shot) | After (agentic) |
|---|---:|---:|
| Answer accuracy, answerable questions | 57.6% | **92.4%** |
| Hallucination rate | 8.6% | **1.2%** |
| Retrieval recall | 59.1% | **95.5%** |
| Context precision (passages on topic) | 28.4% | **99.5%** |
| Replies carrying a citation | 0% | **96.9%** |
| Citation precision | — | **99.5%** |
| Correct refusals on unanswerable questions | 100% | 100% |
| Facts leaked across businesses | 0 | 0 |
| Latency, median | 6.9 s | 13.1 s |
| Model cost per turn | $0.00056 | $0.00109 |

**Where the gain came from.** 25 of the 28 baseline failures were retrieval misses. Most were "store traps": the agent searched the FAQs for a rule that lives in a document. Searching every store at once fixed that whole class (2 of 15 → 15 of 15). Grading kept it affordable: the model saw 1.2 passages per answer instead of 3.6, and 99.5% of them were on topic.

**The trade-off.** Grading and the grounding check are two extra model calls, so median latency roughly doubled. That is fine on WhatsApp, where replies are already batched, and it is the first thing to optimise for the website widget.

## Production practices

- **Tenant isolation** is enforced in one place and tested: 0 cross-tenant leaks in the evaluation.
- **Security:** auth on every route, rate limiting, a guarded internal webhook, OAuth 2.1 for outside AI clients.
- **Testing:** an 81-question retrieval evaluation, 26 end-to-end tests for the MCP server, and test suites for auth, channels, rate limiting and the widget.
- **Safe rollout:** one setting switches retrieval back to single-shot, so the new loop can be rolled back without a deploy.
- **Deployment:** Nginx with TLS in front of FastAPI and a background worker on a Linux server, with Postgres + pgvector and Redis.

## Tech stack

<p>
  <img src="https://img.shields.io/badge/Python-101018?style=for-the-badge&logo=python&logoColor=3BA4F0" alt="Python"/>
  <img src="https://img.shields.io/badge/FastAPI-101018?style=for-the-badge&logo=fastapi&logoColor=3DD68C" alt="FastAPI"/>
  <img src="https://img.shields.io/badge/LangGraph-101018?style=for-the-badge&logo=langgraph&logoColor=9B8CFF" alt="LangGraph"/>
  <img src="https://img.shields.io/badge/OpenAI-101018?style=for-the-badge&logo=openai&logoColor=F5F5F8" alt="OpenAI"/>
  <img src="https://img.shields.io/badge/PostgreSQL%20+%20pgvector-101018?style=for-the-badge&logo=postgresql&logoColor=5AB4F5" alt="PostgreSQL + pgvector"/>
  <img src="https://img.shields.io/badge/Redis%20+%20ARQ-101018?style=for-the-badge&logo=redis&logoColor=F0616D" alt="Redis + ARQ"/>
  <img src="https://img.shields.io/badge/React%20+%20Vite-101018?style=for-the-badge&logo=react&logoColor=5AB4F5" alt="React + Vite"/>
  <img src="https://img.shields.io/badge/Stripe-101018?style=for-the-badge&logo=stripe&logoColor=9B8CFF" alt="Stripe"/>
  <img src="https://img.shields.io/badge/WhatsApp%20API-101018?style=for-the-badge&logo=whatsapp&logoColor=3DD68C" alt="WhatsApp API"/>
  <img src="https://img.shields.io/badge/Nginx-101018?style=for-the-badge&logo=nginx&logoColor=3DD68C" alt="Nginx"/>
</p>

---

<p align="center">
  Built by <a href="https://github.com/ahmed-hussain-ai"><b>Ahmed Hussain</b></a> · <a href="https://ahmedhussain.tech">ahmedhussain.tech</a><br/>
  <sub>The source code is kept in a private repository. Happy to walk through it in an interview.</sub>
</p>
