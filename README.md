<p align="center">
  <img src="assets/hero.svg" alt="Vendra — multi-tenant AI sales agent. 92.4% answer accuracy, 1.2% hallucination, 95.5% retrieval recall, 0 cross-tenant leaks" width="100%"/>
</p>

<p align="center">
  <a href="https://coachsync.ahmedhussain.tech"><img src="https://img.shields.io/website?url=https%3A%2F%2Fcoachsync.ahmedhussain.tech&label=status&up_message=live&down_message=offline&up_color=3DD68C&style=for-the-badge&labelColor=101018" alt="Live status"/></a>
  <img src="https://img.shields.io/badge/Agentic%20RAG-3BA4F0?style=for-the-badge&logo=langchain&logoColor=white&labelColor=101018" alt="Agentic RAG"/>
  <img src="https://img.shields.io/badge/MCP%20server-9B8CFF?style=for-the-badge&logo=anthropic&logoColor=white&labelColor=101018" alt="MCP server"/>
  <img src="https://img.shields.io/badge/OAuth%202.1-3DD68C?style=for-the-badge&logo=auth0&logoColor=white&labelColor=101018" alt="OAuth 2.1"/>
  <img src="https://img.shields.io/badge/Multi--tenant-D06CC0?style=for-the-badge&labelColor=101018" alt="Multi-tenant"/>
</p>

<p align="center">
  <a href="#01"><b>Why</b></a> &nbsp;·&nbsp; <a href="#02"><b>Features</b></a> &nbsp;·&nbsp; <a href="#03"><b>Architecture</b></a> &nbsp;·&nbsp; <a href="#04"><b>Results</b></a> &nbsp;·&nbsp; <a href="#05"><b>Production</b></a> &nbsp;·&nbsp; <a href="#06"><b>Stack</b></a> &nbsp;·&nbsp; <a href="#07"><b>Limits</b></a>
</p>

<br/>

<a id="01"></a>
<img src="assets/s01.svg" alt="01 · Why Vendra" width="100%"/>

**Vendra gives any business its own AI sales agent.** It answers buyers on WhatsApp, in a website chat widget and over Gmail, from that business's own catalog, documents and FAQs. It books meetings, sends Stripe payment links in the chat and hands over to a human at any point. One platform serves many businesses, each fully isolated from the others.

<p align="center"><img src="assets/problem-solution.svg" alt="Without Vendra vs with Vendra" width="100%"/></p>

> [!IMPORTANT]
> A sales agent that invents a price or a policy costs real money. Vendra's rule is simple: **no claim goes out unless a source supports it.** Every answer is retrieved, graded and checked against its sources before it is sent.

<br/>

<a id="02"></a>
<img src="assets/s02.svg" alt="02 · What it does" width="100%"/>

<p align="center"><img src="assets/features.svg" alt="Omnichannel, agentic RAG, grounding check, MCP server, multi-tenant, books and sells" width="100%"/></p>

<br/>

<a id="03"></a>
<img src="assets/s03.svg" alt="03 · Architecture" width="100%"/>

<p align="center"><img src="assets/architecture.svg" alt="Vendra architecture" width="100%"/></p>

<details>
<summary><b>One message, step by step</b> (sequence diagram)</summary>
<br/>

```mermaid
%%{init: {'theme':'dark', 'themeVariables': {'primaryColor':'#181822','primaryTextColor':'#F5F5F8','primaryBorderColor':'#3BA4F0','lineColor':'#9B8CFF','actorBkg':'#181822','actorBorder':'#3BA4F0','noteBkgColor':'#101018','noteTextColor':'#A3A3B5'}}}%%
sequenceDiagram
    autonumber
    actor C as Customer
    participant CH as Channel (WhatsApp / widget / Gmail)
    participant Q as Queue (Redis + ARQ)
    participant A as Sales agent (LangGraph)
    participant R as RAG loop
    participant DB as Postgres + pgvector
    participant GC as Grounding check
    C->>CH: "Is there a free trial class?"
    CH->>Q: buffer message, return at once
    Q->>A: worker runs the turn
    A->>R: search tool call
    loop until relevant passages found
        R->>DB: hybrid search over 4 stores
        DB-->>R: candidate passages
        R->>R: grade each passage, rewrite query if none fit
    end
    R-->>A: graded passages + trace
    A->>GC: draft reply
    GC-->>A: every claim cited? rewrite or fall back
    A->>CH: grounded reply with source
    CH->>C: "Yes, your first demo class is free."
```

</details>

| Part | What it does |
|---|---|
| **Channels** | WhatsApp, a one-script-tag website widget and Gmail feed the same agent. A new channel is one adapter module, and nothing in the agent changes. |
| **Sales agent** | A LangGraph router with specialist agents. It runs on a worker queue, so a web request never waits on an agent turn. |
| **Agentic RAG loop** | Searches all four knowledge stores at once (catalog, FAQs, documents, business profile), grades every passage, and rewrites the query when nothing relevant is found. |
| **Grounding check** | Every claim in a draft must cite a passage. If it can't, the reply is rewritten from its sources or replaced with "the team will confirm". |
| **MCP server** | OAuth 2.1 with PKCE, dynamic client registration, a consent page and refresh-token rotation; 9 tools with per-tool scopes, and every call is audit-logged. |
| **Observability** | Every search is stored as a retrieval trace. The dashboard's *Answer quality* tab shows the sources behind each answer. |

<br/>

<a id="04"></a>
<img src="assets/s04.svg" alt="04 · Measured results" width="100%"/>

<p align="center"><img src="assets/results.svg" alt="Before vs after: accuracy 57.6% to 92.4%, recall 59.1% to 95.5%, precision 28.4% to 99.5%, citations 0% to 96.9%, hallucination 8.6% to 1.2%" width="100%"/></p>

| Also measured | Before | After |
|---|---:|---:|
| Correct refusals on unanswerable questions | 100% | **100%** |
| Facts leaked across businesses | 0 | **0** |
| Citation precision (cited passage is a gold passage) | — | **99.5%** |
| Fabricated citations | — | **0** |
| Passages handed to the model, mean | 3.6 | **1.2** |
| Latency, median | 6.9 s | 13.1 s |
| Model cost per turn | $0.00056 | $0.00109 |

**Where the gain came from.** 25 of the 28 baseline failures were retrieval misses. Most were "store traps": the agent searched the FAQs for a rule that lives in a document. Searching every store at once fixed that whole class (2 of 15 → 15 of 15), and grading kept it affordable: the model saw 1.2 passages per answer instead of 3.6, and 99.5% of them were on topic.

<br/>

<a id="05"></a>
<img src="assets/s05.svg" alt="05 · Built for production" width="100%"/>

| Decision | Why |
|---|---|
| **Agent on a worker + queue** | The API drops each turn on a queue and returns at once. A slow model call never blocks a web request or a webhook. |
| **Tenant isolation in one place** | Written and tested once, never re-implemented per feature. 0 cross-tenant leaks in the evaluation. |
| **Grounding check after generation** | Retrieval quality alone doesn't stop a model from embellishing. Checking the finished draft does. |
| **Retrieval traces stored for every answer** | When an answer is wrong, the trace shows whether retrieval, grading or writing failed. |
| **OAuth 2.1 for outside AI clients** | Other assistants get scoped, revocable, audited access instead of a shared API key. |
| **One-switch rollback** | A single setting returns retrieval to single-shot, with no deploy needed. |
| **Measured, not assumed** | An 81-question evaluation, 26 MCP end-to-end tests, and suites for auth, channels, rate limiting and the widget. |

<br/>

<a id="06"></a>
<img src="assets/s06.svg" alt="06 · Tech stack" width="100%"/>

<p align="center">
  <img src="https://skillicons.dev/icons?i=python,fastapi,react,vite,postgres,redis,docker,nginx,linux&theme=dark" alt="Python, FastAPI, React, Vite, PostgreSQL, Redis, Docker, Nginx, Linux"/>
</p>
<p align="center">
  <img src="https://img.shields.io/badge/LangGraph-101018?style=for-the-badge&logo=langgraph&logoColor=9B8CFF" alt="LangGraph"/>
  <img src="https://img.shields.io/badge/OpenAI-101018?style=for-the-badge&logo=openai&logoColor=F5F5F8" alt="OpenAI"/>
  <img src="https://img.shields.io/badge/pgvector-101018?style=for-the-badge&logo=postgresql&logoColor=5AB4F5" alt="pgvector"/>
  <img src="https://img.shields.io/badge/ARQ%20worker-101018?style=for-the-badge&logo=redis&logoColor=F0616D" alt="ARQ"/>
  <img src="https://img.shields.io/badge/MCP-101018?style=for-the-badge&logo=anthropic&logoColor=D06CC0" alt="MCP"/>
  <img src="https://img.shields.io/badge/Stripe-101018?style=for-the-badge&logo=stripe&logoColor=9B8CFF" alt="Stripe"/>
  <img src="https://img.shields.io/badge/WhatsApp%20API-101018?style=for-the-badge&logo=whatsapp&logoColor=3DD68C" alt="WhatsApp API"/>
</p>

<br/>

<a id="07"></a>
<img src="assets/s07.svg" alt="07 · Honest limits" width="100%"/>

- **Latency roughly doubled** (6.9 s → 13.1 s median) because grading and the grounding check are extra model calls. That is fine on WhatsApp, where replies are batched anyway; it is the first thing to optimise for the website widget.
- **The evaluation is one run on two businesses.** The rewrite step mattered little on this corpus and should matter more on larger, messier ones, which is untested.
- **Aggregation questions** ("which courses are under 40,000 and online?") need a structured catalog query rather than similarity search.

<br/>

<p align="center">
  <a href="https://ahmedhussain.tech"><img src="assets/footer.svg" alt="Built by Ahmed Hussain" width="100%"/></a>
  <br/><sub>The source code is private. Happy to walk through it in an interview.</sub>
</p>
