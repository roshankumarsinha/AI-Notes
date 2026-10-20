# AI Interview Question: LLMs vs. Agents — Explained Simply

**YouTube video:** https://www.youtube.com/watch?v=d-98DZZrS7M
**Channel:** Gaurav Sen (GKCS)

What's the difference between an LLM and an agent?" — and keeps pressing until the full architecture of an agent is drawn out.

## The One-Line Answer (the entry ticket, not the full answer)

- An **agent** can **perform actions** — it can trigger calls to external systems or your own internal ones.
- An **LLM** on its own **cannot** — it only produces text.

## The Running Example: A Trip Planner

A user says: _"I'm going to Austria, please plan a trip itinerary."_ That request reaches your system (called "TripAdvisor" in the example). The naive design is: your system just forwards this to an LLM and asks it to make the plan.

**The problem:** that request is almost empty. From "I'm going to Austria," the system knows nothing about _who this person is_ — are they travelling with family? What are their flight preferences? Hotel budget? Dates?

**This is the key insight:** a plain LLM call doesn't fail because the model can't write a good itinerary — it absolutely can. It fails because **the information needed to make good decisions isn't in the request at all.** What turns an LLM into an agent is a _mechanism for going and getting the information that's missing._

## How an Agent Fills the Gaps

```mermaid
flowchart TD
    U["User: 'Plan my Austria trip'"] --> Sys["Your system (TripAdvisor)"]
    Sys --> LLM["LLM"]
    LLM -->|"1. Missing info!\nAsk user a form"| U
    LLM -->|"2. Decides it needs data"| Sys
    Sys -->|"executes with permissions + keys"| Flight["Flight system\n(prices)"]
    Sys -->|"executes with permissions + keys"| Hotel["Hotel system\n(rooms, prices)"]
    Flight -->|"results back as context"| LLM
    Hotel -->|"results back as context"| LLM
    LLM -->|"3. Now has enough context\n→ makes the plan"| Sys
    Sys -->|"final itinerary"| U
```

Two ways the agent gathers what it's missing:

1. **Counter-question the user.** The first "action" isn't even an external API call — it's the model realizing it lacks information and turning back to the human, e.g. asking them to fill in a form (flights, dates, preferences). The human is just another source the agent can query.
2. **Query the flight and hotel systems.** Once it has the user's details, it gathers live prices and room availability.

**The order matters:** context _first_, decision _second_. The tools don't make the model smarter — they give it the facts it needs before it's asked to decide.

## The Follow-Up That Reframes Everything

The interviewer replays his own words back: _you keep saying "the LLM queries the flight system."_ Then the sharp question:

> **Is the LLM making the call, or deciding to make the call?**

The right answer: **the LLM only _decides_.** It cannot make the API call itself. It emits an _intention_; something else executes it.

This is the same "an LLM can't make API calls" line from the start — but now it means something precise: it's not a vague statement about model limitations, it's about **where the execution boundary sits in your architecture.** The model chooses; your system does.

## Why Your System Sits in the Middle

Since the model only decides, something has to actually execute — and that's your own system, because it holds what the model doesn't:

- **Permissions** — your system has the right to call the hotel/flight systems; the model has no credentials or authority.
- **Rules / context** — a system prompt telling the model _how_ and _when_ to call things.
- **Auth keys** — which live on _your_ side of the boundary, never in the model's context.
- **A return path** — a way to feed results back.

That whole bundle — permissions, rules, keys, return channel — is what "**tool**" actually means here. Not just a function the model can call, but the entire safe-execution wrapper around a decision the model made.

**Real-world example:** think of the LLM as a manager who says "book me a flight," and your system as the assistant who actually has the company card, the login, and the authority to do it. The manager decides; the assistant executes. The manager never touches the credit card.

## What a "Tool" Definition Contains

The best mental model: a tool is **like a function call.** The model's output contains a **tool call** (e.g. `search_flights`), and a proper tool definition describes:

- **Name** — what the tool is
- **Purpose** — what it does
- **Parameters** — what inputs it needs
- **Per-parameter descriptions** — what each input means
- **Allowed value types** — what can be passed in

## How Multiple Tool Calls Are Orchestrated: A Loop

```mermaid
flowchart TD
    Start["User query arrives"] --> Decide{"LLM: do I have\nenough context?"}
    Decide -->|"No, need more"| Call["Emit tool call(s)"]
    Call --> Exec["System executes them"]
    Exec --> Decide
    Decide -->|"Yes - success"| Done["Forward final plan to user"]
    Decide -->|"Can't get it - failure"| Fail["Return 'sorry, can't answer'"]
```

It's a **loop**, and crucially the loop condition is a _model decision_, not fixed program logic. On each pass the LLM decides whether to keep making tool calls or exit. It exits on one of two conditions:

1. **Success** — "I have enough context to answer."
2. **Failure** — "I can't get this, sorry."

Only on success does the plan actually go to the user.

**The concurrency point** (clearly from his software-engineering background): "loop vs. sequential" is a false choice. It's a loop, and _within_ each iteration:

- Independent tool calls (a flight check and an unrelated hotel check) run **in parallel / asynchronously** — no reason to serialize them.
- Dependent calls (the flight choice depends on the hotel choice, or vice versa) run **ordered / synchronously**.

---

# AI Interview Question: Context in LLMs — Explained Simply

**YouTube video:** https://www.youtube.com/watch?v=VWNoSjQ798I
**Channel:** Gaurav Sen (GKCS)

What actually sits inside an LLM's context window — and what do you do when it gets huge?

## The Running Example: Plan a Trip to Japan

A user query — _"plan a trip to Japan"_ — goes into "our system," an agent built with three things inside it: the **LLM**, the **system prompt**, and the **tools**.

So at the very start, what's in the context?

- The **system prompt**
- The **tool descriptions** (the full schemas telling the model what each tool does and how to call it — not just names)
- The **user query**

On that basis, the model plans an action and calls its first tool — say, a hotel tool.

## Watching the Context Grow, Call by Call

This is the heart of the answer: **context is append-only.** Nothing ever leaves — every new decision is made on top of the entire history so far.

```mermaid
flowchart TD
    A["Start:\nsystem prompt + tool descriptions + user query\n(call this 'I1')"] --> B["Model calls hotel tool"]
    B --> C["Context now:\nI1 + hotel info"]
    C --> D["Model calls flight tool"]
    D --> E["Context now:\nI1 + hotel info + flight info"]
    E --> F["...and so on, growing every step"]
```

After each tool call, **two** new things get added to context: the _action taken_ and the _output of that action_. That's exactly why context grows without bound across an agent loop — and it's what sets up the follow-up.

## The Definition He Lands On

> Whatever the model is considering in order to make a decision **is** the context.

The four components:

| Component          | What it is                                              |
| ------------------ | ------------------------------------------------------- |
| **User query**     | The request that started the run                        |
| **System prompt**  | The standing instructions to the model                  |
| **Tools**          | The tool descriptions and schemas available             |
| **Tool responses** | The outputs from each call, accumulated across the loop |

## Follow-Up: What if Context Hits Millions of Tokens?

Scale the example up — plan trips for five different people. Now there are far too many tool calls and far too much accumulated context to just keep in one window. Two options:

### Option 1: Summarize(Compact) + a Memory Layer

Build a summarization system that condenses per person, so you keep five _summaries_ instead of five full histories, stored separately and pulled in as needed. This is an explicit **change to the architecture** — the earlier design had no memory layer at all.

**Why bother? To stop the model hallucinating.** His reasoning: with millions of tokens in context, the model gets confused, and you get hallucinations. So summarizing and offloading to memory isn't mainly about cost or hitting the token limit — it's about keeping the model's working set small enough to stay coherent.

### Option 2: Split Across Multiple Agents

Use a multi-agent setup — e.g. one agent for planning, a separate one for summarization. The context gets divided across agents instead of piling into one window (a separation of concerns).

## What Is Memory? Is It Just a Database?

The clean distinction the whole video is built around:

> **Memory is the stored thing. Context is the runtime memory — what the model actually has in front of it while answering this particular query.**

So the split is **persistence vs. liveness**: memory is durable and lives _outside_ the model; context is assembled fresh per query, and is what the model actually sees.

Rule of thumb he gives: modern context windows are ~1 million tokens. If you're not crossing that, you don't need an external store at all — the context window _is_ your storage. You only reach for memory once you outgrow the window.

**Real-world example:** memory is your filing cabinet; context is the handful of papers you pull out and spread on your desk to work on one specific task. The cabinet holds everything long-term; the desk only holds what you need right now.

## How Something Gets From Memory Into Context

Two examples make it click:

- **Single chat session** (like ChatGPT/Claude): everything within one conversation thread is already in the context — no separate memory needed. The session history _is_ the context.
- **Cross-session** (a healthcare bot): "what did the doctor prescribe last visit?" isn't in the current session, so you have to **extract** that from stored memory and insert it into this run's context.

Retrieval is **demand-driven** — nothing loads just because it exists. The typical chain:

1. The LLM asks a clarifying question ("what are your preferences?")
2. The user points at history ("use my preferences from my last trip")
3. That triggers the system to **extract** those preferences from memory
4. The extracted info gets loaded into the context for this run

## "Why Isn't This Just a Tool Call?"

The sharpest challenge: _if you fetch memory with a call to a database, why call it "memory" and not just a tool call?_

The resolution is a layering one:

- The **retrieval** is indeed a tool call.
- The **memory** is the _store behind it_ — the thing being stored, not the act of fetching.

---

# AI Interview Question: Prompt Caching with LLMs — Explained Simply

**YouTube video:** https://www.youtube.com/watch?v=tGqvoEuoU5M
**Channel:** Gaurav Sen (GKCS)

The question: your platform gets repeated prompts — how would you add caching?\*

## The First Move: Clarify Before Designing

Gaurav doesn't jump to an architecture. His first response is a **clarifying question**, and the whole answer hinges on it:

> Are the repeated queries **exact identical text**, or just **similar queries** that deserve similar answers?

**Real-world example:** two users wanting the same trip — one types "going from Bangalore to Mysore," the other types "I stay in Bangalore, going to Mysore for a vacation." Same intent, completely different strings. But if the input comes from _form clicks_ rather than free typing, the exact same string really does arrive every time.

This matters because the answer is totally different depending on the traffic: **exact-text traffic wants a hashmap; paraphrase traffic wants a vector index.** Guessing means building the wrong cache. The interviewer picks the exact-text case first.

## Stage 1: Exact-Match Caching (the baseline)

Drop a cache between your system and the LLM, so every request checks the cache before it's allowed to hit the model.

```mermaid
flowchart TD
    Q["User query arrives"] --> C{"In cache?"}
    C -->|"Hit"| Serve["Serve stored response\n(no LLM call)"]
    C -->|"Miss"| LLM["Call the LLM"]
    LLM --> Store["Store: key = query text,\nvalue = response"]
    Store --> Serve2["Return response to user"]
```

The walkthrough: first user asks "I am going to Mysore from Bangalore" → cache is empty (miss) → LLM generates a plan → store it (key = query text, value = plan). Later, another user produces the identical string → cache hit → serve the stored plan, **no LLM call**. Mechanically, it's just an ordinary key-value cache with exact-equality keys.

### The staleness problem (he raises it himself)

What if the cached answer goes stale? His example: a strike shuts the Bangalore–Mysore road — the cached travel plan is now wrong, and the cache keeps confidently serving it. Two remedies, and a rule for choosing:

- **TTL / timeout** — cached response valid only for a short window. Use this **if the world can change under you.**
- **LRU eviction** — if staleness isn't a real risk in your domain, skip time-based invalidation; just evict for capacity (least-recently-used) reasons.

The choice is driven by **whether staleness is a live risk**, not by cache mechanics.

## Stage 2: Semantic Caching (when meaning matches but text doesn't)

Now the interviewer removes the guarantee: the prompts _mean_ the same thing but aren't identical text. An exact-match cache misses on _every_ rephrasing, so the hit rate collapses. The fix is a **semantic cache**.

The single component that changes: the **key** is no longer the query text — it's the query's **embedding vector** (a mathematical representation of what the query means).

|                | Ordinary key-value cache    | Semantic cache                            |
| -------------- | --------------------------- | ----------------------------------------- |
| **Key**        | the query text              | the embedding vector of the query         |
| **Comparison** | exact string match          | nearest-vector search                     |
| **Result**     | 1 on the match, 0 elsewhere | a ranked list by similarity               |
| **Structure**  | hashmap                     | approximate nearest neighbour (ANN) index |

**How retrieval works:** convert the incoming query to a vector, find the nearest stored vector. Such a search _can_ return the top 5 / 3 / 2 — but for caching you take only the **top 1**, the single closest match, and serve its value. (Unlike RAG, a cache has no downstream reasoning step to weigh multiple candidates — it either answers or it doesn't.)

**The threshold** is the tuning knob: set it (his example: cosine similarity > **0.9**). Above it → treat as the same query, serve the cached answer. Below it → miss, go to the LLM. Too low and you serve wrong answers; too high and you're back to near-exact matching and lose your hit rate.

## The Tradeoffs of a Semantic Cache

A semantic cache introduces **more points of failure**, paid in three currencies.

### 1. Embeddings lose information → inexact answers

Compression is the whole point of an embedding — and what gets thrown away isn't chosen with your use case in mind. His killer example: **"laptop for 3,000"** vs. **"laptop for 1,000."** Those two vectors are nearly identical — same topic, phrasing, and structure — so they can clear the 0.9 threshold and be treated as the same query. Result: the person who asked for a 3,000 machine gets 1,000 recommendations.

**Why this example is the point:** the single detail that carries the entire practical meaning — the price — is exactly the kind of thing embedding compression flattens. Raising the threshold doesn't reliably fix it, because the queries genuinely _are_ that close in vector space.

### 2. Latency: vector search is slower than a hashmap

The search is **approximate nearest neighbours (ANN)** — much slower than an exact key comparison, because an exact comparison is literally a hashmap lookup and this isn't. So hold both directions in mind: a semantic cache is **slower than an exact-match cache, but still faster than an LLM call.**

### 3. Cost: why the cache path still beats the model path

If the uncached path is one LLM call, and the cache also does work — where's the saving?

```mermaid
flowchart LR
    subgraph Cache["Cache path (cheap + fast)"]
        E["Embedding model\nsmall, fast, cheap"]
    end
    subgraph Model["LLM path (expensive + slow)"]
        A["Full LLM inference\nruns through every attention block\n+ provider charges a premium"]
    end
```

- The cache path runs an **embedding model** — small, fast, cheap.
- The LLM path runs the input through **every attention block** — far more computation — and you're usually paying a **provider premium** on top.

**Net position:** an embedding + ANN lookup is _cheaper and faster than an LLM call_, but _slower than an exact-match hashmap_, and it carries a correctness risk the hashmap doesn't.

---

# AI Interview Question: Evals in AI — Explained Simply

**YouTube video:** https://www.youtube.com/watch?v=dzVogKf9jIM
**Channel:** Gaurav Sen (GKCS)

The setup: the interviewer plays a hiring manager at a healthcare company, and the candidate has to explain, live on a whiteboard, how you'd build an AI product reliable enough for a domain where a wrong answer means getting sued.

**The core difficulty in one line:** _AI is non-deterministic._ Classical software gives the same output for the same input, so you can pin it down with tests that assert exact equality. An LLM won't — so reliability has to be established some other way.

## The Central Equation

> **Reliability = Observability + Evals**

The argument is about _timing_: if you can guarantee both **before** production (during testing), you can call the system reliable — and only then is it fit for high-risk areas like healthcare or finance. Reliability here isn't something you bolt on at runtime; it's established in advance, by instrumenting and then measuring.

```mermaid
flowchart LR
    O["Observability\nrecord everything the agent does\n(agent traces)"] --> E["Evals\nmeasure quality against those traces"]
    E --> R["Reliability\nsafe to ship to high-risk domains"]
```

_(A fair framing challenge from the comments: observability arguably already includes evals, so calling them two separate pillars is a bit artificial. The video treats them as separate-but-coupled — observability produces the data, evals consume it.)_

## Observability: Agent Traces, Not Logs

An agent has many moving parts (system prompt, tools, decisions), so the artifact you store is no longer a **log** — it's an **agent trace**, which captures:

1. The **changes in your system** caused by tools (state that moved because a tool ran)
2. **Which tools** are being called
3. The **response** from each tool
4. The **decision the LLM makes** based on that tool response

Item 4 is what makes a trace different from a normal log: you're recording the model's _reasoning step_, not just that something happened.

## What Evals Are: QA for Agents

The analogy: classical software had unit tests plus a separate QA team. **Evals are the QA layer for agents** — but instead of testing the app as a whole, you test the individual nuts and bolts: is the system prompt leaking? Is information leaking during a tool call? Is the right _number_ of tool calls happening? Is a tool being called over and over despite returning the same thing each time?

## The Three Core Metrics

Draw the minimal agent — an LLM, its tools, and a system prompt — then ask what you most want to be true. Three requirements, mapping one-to-one onto three metrics:

```mermaid
flowchart TD
    R1["Want: the output is correct & relevant"] --> M1["Metric 1:\nAnswer Relevancy"]
    R2["Want: the answer uses the context you supplied"] --> M2["Metric 2:\nGroundedness"]
    R3["Want: the tool calls are sane, not repeated needlessly"] --> M3["Metric 3:\nTool Sequence Accuracy"]
```

### Metric 1: Answer Relevancy

Does the agent's response actually address _this_ query, and not some other question? Trivial example: ask what an apple is, and it shouldn't tell you about a banana.

### Metric 2: Groundedness

Does the answer actually use the **context that was supplied**, rather than making things up? The plain-language version: _the agent should not be hallucinating._ Operationally, **groundedness ≈ hallucination measurement.** It's about faithfulness to the supplied material, not truth in the world.

### Metric 3: Tool Sequence Accuracy

Did the agent call its tools in a sane order? His flight-booking example, with three tools:

| #   | Tool                   | Purpose                     |
| --- | ---------------------- | --------------------------- |
| 1   | Get user data          | Retrieve the user's details |
| 2   | Web search for flights | Find available flights      |
| 3   | Payment gateway        | Take payment                |

The failure to catch: the agent jumps straight to searching flights _without first fetching user data_ — or worse, calling **payment before identity**. The ordering itself is part of correctness. You measure it by defining the correct sequence up front, running the agent, and checking whether it matched.

## Who Does Evals? (a surprisingly strong org argument)

In classical dev, QA was a **separate team** that received an app and tested it. With agents, **that separation breaks** — whoever _built_ the system should test it.

## Where the Test Set Comes From (when there's no ground truth)

The missing ingredient vs. classical dev: with an LLM app, **most of the time you have no ground truth** — no correct-answer reference exists for your task. So you manufacture one, two ways:

- **Synthetic dataset** — generated from what you believe to be right, following a pattern.
- **Golden dataset** — built with a **subject matter expert** (in healthcare, a clinician).

The golden dataset is the most important part of the answer: **the reliability of your evals is bounded by the quality of the dataset a domain expert helped you build.**

## How Each Metric Is Actually Computed (the required inputs)

The pattern is **cumulative** — each metric needs everything the previous one needed, plus one more thing:

| Metric                     | Inputs needed                                                        |
| -------------------------- | -------------------------------------------------------------------- |
| **Answer Relevancy**       | user query + response _(optionally + system prompt)_                 |
| **Groundedness**           | user query + response + system prompt + **the context**              |
| **Tool Sequence Accuracy** | the **tool calls made by the assistant** (straight from your traces) |

**Answer relevancy** minimally needs the exact query and the response. Optionally add the system prompt, since instructions can legitimately shape what counts as on-topic.

**Groundedness** adds **the context** — whatever the bot followed to answer. This maps cleanly onto both architectures:

- **In a RAG pipeline:** the context is the **retrieved chunks** (e.g. your top-5).
- **In an agent:** the context is the **tool responses** — literally how an agent acquires context.

A high groundedness score means the agent is faithfully following the supplied context/tool responses — again, faithfulness, not real-world truth.

**Tool sequence accuracy** needs nothing new — just the assistant's tool calls that observability already captured.

---

# AI Interview Question: RAG or Agents? — Explained Simply

**YouTube video:** https://www.youtube.com/watch?v=NqOPcNYbqrA
**Channel:** Gaurav Sen (GKCS)

The question: you're building an app that books flights and hotels and produces a travel plan — should you build a **RAG** pipeline or an **agentic** one? The real object of the interview is the "why not RAG?" — could you build this with RAG, and if not, why?\*

## What RAG Actually Is

Retrieval-Augmented Generation means: when you give the LLM context to answer a query, you don't rely on the model's own trained-in knowledge — you **fetch** that context from a data store (usually a vector database).

```mermaid
flowchart LR
    Q["User query"] --> E["Make an embedding of it"]
    E --> V["Search vector DB\nfor similar stored embeddings"]
    V --> C["Pick the matching text chunk"]
    C --> L["Append to query, hand to LLM"]
    L --> A["Answer"]
```

## What Makes an Agent Different

Two things, and the travel use case needs both:

1. **The loop.** The LLM doesn't get all the context in one shot — it performs multiple actions itself and, at some point, _decides on its own_ that it now has enough to answer. That self-judged stopping point is part of the definition.
2. **Actions that change the world.** Some agent actions don't just _read_ — they _mutate state_: actually making a booking, logging you in, changing what you see next. An LLM alone can't do this; an agent can, because it can make tool calls.

**The one-line contrast:** with RAG, all you can do is **read**. With an agent, you can do more than read, _and_ there's a loop. Travel needs both — you fetch from multiple live sources _and_ you take real actions — so **agents are the right fit.**

## Follow-Up 1: "What if I just add a loop to my RAG?"

The strongest challenge: _if the retrieved context isn't enough, the LLM tells me to retrieve again, I loop. Doesn't that close the gap?_

The answer: **the loop isn't the real difference — the data is.**

- **Ownership & staleness.** RAG operates over static data _you own_. But you don't own the flight data or hotel inventory — you can't store it (and may not even be allowed to). You genuinely have to fetch it **live** at request time.
- **Decision-making.** An agent's LLM decides _what to do next_. His example: if few flights are available and prices are high, the model can decide on its own to switch from one source to another looking for cheaper options — a branch nobody hard-coded. In RAG, the flow is **predefined**: fixed sources, fixed query method, fixed response shape.

**The formulation that carries the whole answer:**

> In RAG, the LLM is an **answer compiler** — it assembles a response from context it was handed.
> In an agent, the LLM is a **decision maker** — it chooses the path. That's why it's called the "brain."

The interviewer plays it back: _with agents I get real-time information; with RAG the information is static/stored and I can only extract what's already there._

## Follow-Up 2: Is "Agentic RAG" a Real Thing?

Is "agentic RAG" a combination, or something new? His answer: it's a **combination** — an agent sits in the middle fetching real-time context via tool calls, and it _can also_ query a vector DB, in which case **the vector-DB lookup just becomes one more tool call** among others. You drop the predefined retrieval path and get a dynamic, per-turn choice of what context to fetch.

```mermaid
flowchart TD
    A["Agent (the brain)"] --> T1["Tool: live flight search"]
    A --> T2["Tool: live hotel search"]
    A --> T3["Tool: vector DB lookup\n(just another tool call)"]
    A --> D["Decides per turn\nwhat context to fetch next"]
```

## Quick Comparison

|                | RAG                  | Agent                                     |
| -------------- | -------------------- | ----------------------------------------- |
| **LLM's role** | Answer compiler      | Decision maker ("the brain")              |
| **Can do**     | Read only            | Read + write actions                      |
| **Data**       | Static, owned by you | Live, fetched at request time             |
| **Flow**       | Predefined path      | Self-directed loop, stops when it decides |

---

# AI Interview Question: 1M Context Window, or RAG? — Explained Simply

**YouTube video:** https://www.youtube.com/watch?v=inYutMr6fN4
**Channel:** Gaurav Sen (GKCS)

The problem: a customer-support app with ~100,000 users keeps getting more and more documents added — how do you scale it?

## The Problem

A support bot already serves ~100,000 users. As the company grows and questions change, more and more documents pile into the support pipeline. How do you handle all these documents and keep scaling as the set grows?

Answer: a **RAG (Retrieval-Augmented Generation)** pipeline, whose central move is **chunking** large documents into small pieces.

## The Two Halves of RAG

```mermaid
flowchart TD
    subgraph Offline["Offline: ingestion (done once, ahead of time)"]
        D["Company documents"] --> CH["Chunk into small pieces"]
        CH --> EM["Convert each chunk to an embedding"]
        EM --> VDB["Store embeddings in a vector DB"]
    end
    subgraph Online["Online: at query time"]
        Q["User query:\n'How many leaves do I have?'"] --> QE["Embed the query\n(same embedding model)"]
        QE --> S["Search vector DB\nfor the closest chunks"]
        S --> K["Take top-K chunks"]
        K --> G["Send chunks + query to LLM"]
        G --> A["LLM writes the answer"]
    end
    VDB -.-> S
```

## Step 1: Chunking

The old problem was that the documents were too big to fit in the LLM's **context window**. The fix is to break each document (e.g. a PDF) into smaller pieces that _do_ fit. Typical chunk size: **800–1,000 characters**.

## Step 2: Embeddings — and Why a Vector DB, Not Postgres

Each chunk becomes an **embedding** — a numerical representation of its meaning. The interviewer pushes back: _we already use Postgres, and SQL has indexes too — why add a vector database?_

Answer is **dimensionality**:

- A SQL index compares along basically **one dimension**.
- An embedding has **many** — one chunk might be a **768-dimension** vector, and some models go up to **8,192 dimensions**.

A vector DB is purpose-built to compare these high-dimensional vectors and find the closest ones; a regular SQL index isn't.

**Real-world example:** matching people by a single trait (say, height) is easy to sort and index. Matching them by _overall similarity_ across hundreds of traits at once is a completely different kind of search — that's what a vector DB does.

## Step 3: The Query Path

When an employee types _"How many leaves do I have?"_ and hits enter:

1. The **query is embedded** — turned into a vector.
2. Using the **same embedding model** that embedded the chunks (mismatched models would make the comparison meaningless).
3. The query vector is used to **search the vector DB** for the chunks most likely to answer it, using an **approximate-nearest-neighbour (ANN)** algorithm like **HNSW**.

## Step 4: Choosing K (how many chunks to retrieve)

**K** is how many chunks you pull back — usually start around **5 or 10**, then tune empirically.

## Step 5: Generation — and Why This Cuts Hallucination

Previously, _all_ the documents were sent to the LLM. RAG's final step looks similar but differs in **what goes into the prompt**: only the retrieved chunks (the **context**) plus the user's query. The LLM's only job is to write the answer from that.

**Why it reduces hallucination:** with the old setup, the documents were too big to fit, so the LLM lacked proper context and could **make things up or miss details**. Feeding it just the _relevant_ chunks means it answers from real supplied material instead of inventing it.

_(She notes that forcing the LLM to answer_ only _from the supplied context is a separate problem, handled in **evals** — and leaves it there.)_

## Follow-Up: Real PDFs Have Tables, Screenshots, and Scans

The interviewer complicates it: the data is mostly **PDFs with tables** — currency-conversion rates, policy rules ("if more than 7 days have passed, do X, else Y"), and sometimes tables that are just **screenshots/images** pasted in (or fully **scanned** pages). Tables can even be **hierarchical** (several rows grouped under one parent row). How do you extract that?

Her key insight: **only the ingestion pipeline needs to change — retrieval and generation stay exactly the same.**

### The Fix: Layout Detection + a Parser Per Element Type

Plain PDF parsers like **PyPDF** or **PyMuPDF** generally don't handle tables well, so:

```mermaid
flowchart TD
    P["PDF page"] --> LD["Layout-detection step\nlabels each region"]
    LD --> R1["Logo / title"]
    LD --> R2["Table region"]
    LD --> R3["Text region"]
    R2 --> TP["Table parser"]
    R3 --> TXT["Text parser"]
    TP --> M["Merge into clean chunks → embed → store"]
    TXT --> M
```

1. **Detect the layout** — an algorithm scans each page and labels what's where ("this region is a table," "this is text," "this is the logo").
2. **Route each region to the right parser** — one parser for text, a different one for tables.
3. **Why split them:** a single parser usually can't extract all these different element types well.

**Real-world example:** like sorting mail before opening it — letters, parcels, and magazines each go to a different handling process, because one method can't deal with all three properly.

---

# AI Interview Question: Why Is This Chatbot Failing? — Explained Simply

**YouTube video:** https://www.youtube.com/watch?v=dob12Lp2HZM
**Channel:** Gaurav Sen (GKCS)

The interviewer describes a RAG chatbot already in production whose answers aren't accurate enough. The candidate has to work out what's wrong on a whiteboard and fix it.\*

## The Problem

The team already has a RAG pipeline in production, but it isn't reaching the accuracy they want. The question: **what would you change in the architecture to fix it?**

## Narrowing Down Where It Breaks

When RAG accuracy is low there are a few usual suspects, so he checks them in order and rules each one in or out:

| Suspect                                          | Interviewer's answer                       | Verdict                 |
| ------------------------------------------------ | ------------------------------------------ | ----------------------- |
| Are the most relevant chunks being retrieved?    | "Yes, that's exactly where the trouble is" | **The problem is here** |
| Offline side (ingestion/indexing)                | Already checked, works fine                | Ruled out               |
| Online side (what happens when a query comes in) | This is where it fails                     | **Focus here**          |
| Query rewriting before retrieval                 | Not used, not needed for their case        | Ruled out               |
| Weak embedding model                             | Possible but unlikely                      | Low suspicion           |

**His working hypothesis:** the similarity search marks chunks as relevant to the query, but many of them aren't actually relevant.

## The Culprit: Top-K Is Too Large (20–25 Chunks)

He asks how many chunks they retrieve per query (the **top-k**). The answer is **20–25**.

He calls that a lot, and explains why it hurts:

- 20–25 chunks fill up the LLM's context window.
- The LLM then has to dig through all of it to find what matters, a **"needle in a haystack"** problem.
- Performance drops, and it can start to **hallucinate**.

Why did they pick 20? They decided it empirically, because they create a lot of chunks. His guess at the real story is a common trap: **they weren't getting the relevant chunks, so they kept raising top-k to dig deeper, and that polluted the context.**

## The Fix: Retrieve Wide, Then Rerank and Filter

Simply cutting top-k back to 10 risks losing the relevant chunks again. So he takes both sides of the problem:

```mermaid
flowchart LR
    Q["User query"] --> V["Vector search\nretrieve 20-25 chunks\n(cheap, wide net)"]
    V --> R["Reranker\nre-score each chunk\nagainst the query\n(expensive, precise)"]
    R --> F["Keep top ~10\ndrop the rest"]
    F --> L["LLM writes the answer"]
```

1. **Keep retrieving 20–25** so the relevant chunks still get a chance to show up.
2. **Add a reranking step**: re-check each retrieved chunk against the query and score it again.
3. **Filter down to about 10** of the best chunks.
4. Only those go to the LLM.

**Real-world example:** hiring. A recruiter quickly shortlists 25 CVs by keyword match (cheap and wide). A senior engineer then reads each of the 25 carefully against the actual job (slow and precise) and picks the top 10 for interviews.

## Why Embedding Similarity Gets It Wrong

The vector database turns the query into a vector (call it **V1**) and finds the stored chunk vectors (V100, V101, V102…) closest to it. The weakness is that **some kinds of meaning are very hard to capture in an embedding:**

| Example                                                                     | Why embeddings struggle                                     |
| --------------------------------------------------------------------------- | ----------------------------------------------------------- |
| "Laptop between Rs 300 and Rs 400"                                          | Numeric ranges are very hard to embed                       |
| "This is **not** what I want" vs. "This is what I want"                     | Opposite meanings, but the vectors land very close together |
| "This is an aircraft **carrier**" vs. "This is an aircraft **part**"        | Different things, potentially very close in embedding space |
| "I **don't** want a 7-day refund period" vs. "I want a 7-day refund period" | Overall wording is near-identical, so the vectors are too   |

A human knows the word **"not"** flips the whole meaning. An embedding squeezes the sentence into one vector, and that small word can get lost.

## How a Reranker (Cross-Encoder) Works

The reranker does **token-level matching** instead of comparing two separately made vectors. Internally it's a **cross-encoder**:

```mermaid
flowchart TD
    Q["Query tokens"] --> CE["Cross-encoder\nreads query + chunk together"]
    C["One chunk's tokens"] --> CE
    CE --> S["Relevance score\nfor this (query, chunk) pair"]
```

1. The query is **not** turned into its own separate embedding.
2. The query and one chunk are fed into the model **together**, so it can see how each word in the query relates to each word in the chunk (including words like "not").
3. It outputs **one score** for that pair.
4. Repeat for every retrieved chunk. For 20–25 chunks you get 20–25 scores, one each.
5. Keep the highest-scoring ~10 and drop the rest.

Expected result: better accuracy and usually fewer hallucinations.

**Why "together" matters:** a normal embedding model (a **bi-encoder**) summarises the query and the chunk separately and then compares the two summaries. A cross-encoder reads them side by side, like comparing two documents line by line instead of comparing their one-line abstracts.

## "Why Not Just Use the Reranker for Everything?"

The interviewer's sharpest challenge: if the reranker is that good, why use the vector DB at all?

The answer is **scale**. It's a deliberate two-step design:

- **Step 1 (cheap):** embed the query and find nearest neighbours. This works even across millions of chunks, because the chunk vectors were computed ahead of time.
- **Step 2 (expensive):** the cross-encoder runs a full model pass for every (query, chunk) pair. Doing that for every chunk in the corpus, on every query, would be far too slow and costly.

So the cheap step narrows the field, and the expensive step only runs on the small shortlist.

## The Trade-off: When Is a Reranker Worth It?

Reranking adds an extra stage to every request, so it costs latency and money. His framing:

- If the reranker costs something like **50–100 ms** and you're willing to pay that for the accuracy gain, it's a good idea.
- Expect roughly a **5–10% accuracy improvement** (an illustrative figure, not a guarantee).
- **Prove it with metrics.** He names **faithfulness** and **answer relevancy** (the same eval metrics from the Evals episode). If those go up, the extra latency and cost are justified.

## Worth Knowing (added context)

There's one more retrieval trick that can be used in conjunction with a reranker:

- **Hybrid search**: combine vector search with keyword search (BM25). Keyword search catches exact terms, product codes and specific words that embeddings blur, and the two result lists get merged. This is often used _together with_ a reranker.

---

# AI Interview Question: Why Is the Chatbot So Slow? — Explained Simply

**YouTube video:** https://www.youtube.com/watch?v=Tb5iXgqflHI
**Channel:** Gaurav Sen (GKCS)

Customers complain the company chatbot is too slow and wants it to "respond in milliseconds."

## The Ask, and the First Pushback

The manager wants responses in **milliseconds**. The candidate's first move is to set expectations honestly: that's hard to promise. If the bot calls an **external LLM API**, part of the total time depends on a system the team doesn't control. He commits to making it as fast as possible instead.

## Diagnosing Where the Time Goes

Instead of guessing, he narrows it down step by step:

```mermaid
flowchart TD
    A["Chatbot is slow"] --> B{"RAG or agent?"}
    B -->|"RAG, answers from company docs"| C{"Do users upload documents?"}
    C -->|"No, chat only"| D["Ingestion is NOT the cause\nFocus on inference (answering)"]
    D --> E{"Retrieval or LLM generation?"}
    E -->|"LLM is fast"| F["Retrieval is the suspect"]
    F --> G{"Extra steps?"}
    G --> H["No query rewriting\nOne reranker"]
    F --> I["Prime suspect:\nvector DB search"]
```

| Question                         | Answer                              | What it rules out                      |
| -------------------------------- | ----------------------------------- | -------------------------------------- |
| RAG-based or an agent?           | RAG, answers from company documents | Agent loops, tool calls                |
| Do users upload their own files? | No, they just chat                  | Ingestion (parsing/chunking/embedding) |
| Is the LLM slow to generate?     | No, it's fast                       | Generation                             |
| Query rewriting? Reranking?      | No rewriting, one reranker          | Rewriting                              |

**What's left:** retrieval, i.e. fetching chunks from the vector database.

## The Likely Culprit: Vector Database Search

The query is turned into an **embedding**, and that has to be matched against stored chunk vectors to find the nearest ones. This is **approximate nearest-neighbour (ANN)** search, using algorithms like **HNSW** or **IVF**. If they aren't tuned well, the search is slow.

**How big is it?** More than **1 million documents**, each split into about **5–10 chunks**, so roughly **10 million vectors** searched on every query.

## His Fixes for Slow Retrieval

1. **Tune the ANN algorithm** (HNSW/IVF settings).
2. **Run the search in parallel.**
3. **Handle user queries in parallel** (probably already done).
4. **Cache with Redis**: keep recent or commonly-hit documents in a fast in-memory cache. Redis supports vector search, so common queries can be answered without hitting the big database.
5. **Shard the vector DB**: split it into several smaller databases by document type/topic and **route each query to the right shard**, so you search a slice instead of everything.

## Corrections and Added Context

**1. The reranker was left off the suspect list.** They said they have **one reranker**, and from Episode #7 a cross-encoder reranker typically adds roughly **50–100 ms** per request. That's a real, measurable cost and should be checked alongside the vector search, not skipped.

**2. 10 million vectors isn't huge for a well-set-up vector DB.** The video implies that even an optimised search will be slow at this scale. Not necessarily: HNSW search time grows very slowly with the number of vectors, and a well-tuned index on 10 million vectors can usually answer in **a few to tens of milliseconds** when it fits in memory. If it's slow, the usual reasons are:

- the index **doesn't fit in RAM**, so it's reading from disk;
- search settings are set for very high accuracy (e.g. a large `ef_search` in HNSW), which trades speed for recall;
- there's **no proper ANN index**, so it's doing a brute-force scan.

**3. Redis caching overlaps with semantic caching.** Caching common queries is exactly the idea from Episode #3 (Prompt Caching). Caching the **final answer** for repeat or similar questions skips retrieval _and_ the LLM, which is a bigger saving than caching documents alone.

**4. Streaming helps the manager's real complaint.** Streaming the LLM's answer word by word doesn't make the total time shorter, but users see text in well under a second, so the bot **feels** much faster. Since the complaint is about customers' experience, this is often the cheapest win.

## Hypothetical: What If Document Upload Were Slow?

The interviewer switches scenarios. The ingestion pipeline, in order (he corrects himself that parsing comes first):

```mermaid
flowchart LR
    P["Parse the document"] --> C["Chunk it"]
    C --> E["Embed each chunk"]
    E --> S["Store in vector DB"]
```

Where it can be slow:

- **Parsing** is often slow, and can also be inaccurate when you don't know the document format in advance.
- **Chunking**, especially with a **semantic chunker**, which they do use.

### Why Semantic Chunking Is Slow

A normal chunker just cuts text every N characters. A **semantic chunker** tries to cut where the topic changes. To do that, it **embeds sentences and compares the similarity of neighbouring pieces**, repeatedly, to decide where the boundaries go. That means many extra embedding calls and comparisons per document, which adds up fast across millions of documents. He calls it "great in theory, doesn't really work in practice at scale."

**Real-world example:** cutting a long roll of paper into equal pieces with a ruler is quick. Reading it and cutting only at the end of each topic is better quality, but much slower.

---

# AI Interview Question: Why Is the Agent So Slow? — Explained Simply

**YouTube video:** https://www.youtube.com/watch?v=Lck0xGxMBRk
**Channel:** Gaurav Sen (GKCS)

\*Episode 9 of the mock-interview series, and a follow-on to Episode #8 (the slow chatbot). This time it's an **agent**, not a RAG chatbot. A company uses an AI agent to help with employee performance reviews, but every query takes so long that people struggle to use it.

## The Problem

The company's performance-review AI is too slow: every new query takes a very long time.

It's an **agentic application**. You can't chat with it back and forth, but you give it prompts and it goes off and does work using tools.

## Pinning Down the Architecture First

Before guessing, he builds a picture of the system:

| Question                       | Answer                                                               |
| ------------------------------ | -------------------------------------------------------------------- |
| What data does it need?        | Employee data, Git commits, probably some documentation              |
| How many tools?                | About **five** (call them T1–T5), with an LLM deciding which to call |
| LLM via API or self-hosted?    | **API calls**                                                        |
| Memory layer, or single agent? | **Single agent**, no separate memory, no stored preferences          |

So everything the agent knows comes from what's put into its context for that run (pure "context engineering").

## Problem 1: Tools Run One After Another (Synchronously)

1st Problem can be every tool call runs **synchronously**: the agent calls T1, waits for the answer, then calls T2, waits, and so on. With five tools, all those waits stack up.

```mermaid
flowchart LR
    subgraph Sync["Synchronous: waits stack up"]
        direction LR
        A1["T1"] --> A2["T2"] --> A3["T3"] --> A4["T4"] --> A5["T5"]
    end
```

```mermaid
flowchart LR
    S["Start"] --> B1["T1"]
    S --> B2["T2"]
    S --> B3["T3"]
    S --> B4["T4"]
    S --> B5["T5"]
    B1 --> E["All done\n(total ≈ slowest tool)"]
    B2 --> E
    B3 --> E
    B4 --> E
    B5 --> E
```

**The fix:** run **independent** tools at the same time (asynchronously). Then the total wait is roughly the time of the _slowest_ tool, not the sum of all five.

## Problem 2: The MCP Server Exposes Too Many Tools

Their tools come from **MCP servers** (GitHub, plus employee data exposed over MCP). One company-wide MCP server might offer **10 tools** when this agent only needs **5**.

His fix: **filter the tool list** and only give the agent the tools it actually needs. Two benefits from one change:

1. **Less context bloat:** every tool's description goes into the prompt, so fewer tools means a shorter prompt.
2. **Lower latency:** a shorter prompt is quicker to process, and the model has fewer options to choose between.

## When One Tool Is Especially Slow: Cache the Stable Data

Say **T3** (fetching employee data) is the slow one. Much of that data **barely changes day to day**, like an employee's number of leaves. So:

- **Pre-compute and store** those values in the backend ahead of time.
- **Put them straight into the agent's context** (e.g. in the system prompt) at the start of the run.
- The agent no longer needs to call T3 at all for that information.

## Do You Even Need All Five Tools?

Some tools may only fetch routine information that could just be placed in context up front. If so, drop the tool and skip the call-wait-respond cycle entirely.

## Composing (Merging) Tools That Overlap

The interviewer asks: if T2 always waits on T1, can you **merge them into one tool**? Tanishq's rule:

| Situation                                                                           | Fix                                           |
| ----------------------------------------------------------------------------------- | --------------------------------------------- |
| Two tools' jobs **overlap heavily** (e.g. both fetch closely related employee data) | **Merge** them into one tool                  |
| Tools are **independent** of each other                                             | Don't merge; run them **in parallel (async)** |
| Tools are **dependent** and also overlap a lot                                      | **Merge** them too                            |

**Real-world example:** if you always pick up bread and then milk from two shops next door to each other, it's faster to find one shop that sells both.

## Corrections and Added Context

**1. The biggest hidden cost is the LLM round trips, not just the tools.** In an agent loop, every tool call means: the LLM decides → the tool runs → the result goes back → the LLM decides again. Since they call the LLM **over an API**, each of those decisions is a full network call plus generation time. Five tools called one at a time means at least **five or six LLM calls** per query. That's usually more time than the tools themselves.

```mermaid
flowchart LR
    L1["LLM call"] --> T1["T1"] --> L2["LLM call"] --> T2["T2"] --> L3["LLM call"] --> D["...and so on"]
```

So the strongest fixes are the ones that **cut LLM turns**: letting the model request several tools in **one turn** (parallel tool calling, which most model APIs support), merging dependent tools so one call replaces two, and putting stable data into context so some calls never happen. This is also the real reason composing tools helps.

**2. Batching doesn't require multi-agent.** A single agent can batch too, and performance reviews are a perfect fit. If the agent needs data for 20 employees, a tool designed as `get_employee_data(ids=[...])` fetches all 20 in **one** call instead of 20 separate calls (and 20 LLM turns). Your backend can also batch requests from many users of the same single-agent app.

**3. Caching stable data pairs nicely with prompt caching.** If shared, unchanging content (instructions, tool descriptions, company policies) sits at the **start** of the prompt, the LLM provider can reuse its internal work for that prefix on every request (the "prefix caching" from Episode #3), making each call cheaper and faster. Put per-employee data _after_ the shared part, or it breaks the shared prefix. And as in Episode #3, cached employee data needs an expiry (TTL) so it doesn't go stale.

**4. Measure first** (from Episode #8): trace each LLM call and each tool call to see where the seconds really go before changing anything.

---
