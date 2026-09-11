# Module 1: What is the Claude Platform?

## What is the Claude Platform?

### 1. The one-line idea

**The Claude Platform is how you use Claude from your code instead of from a browser.**

When you chat with Claude on the website, _you_ type and _you_ read the answer.
When you use the Claude Platform, _your program_ sends the question and _your program_ receives the answer — so you can do something with it automatically.

> **Simple example:**
> Chat = you copy a customer email into Claude, read the reply, paste it back into your help desk.
> Platform = your help desk app does all three steps by itself when someone clicks a button.

### 2. What you actually get

The platform is not one single thing. It's a small toolbox:

| Piece        | In plain words                                                | Example                                                                    |
| ------------ | ------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **REST API** | A web address you send a request to. Works from any language. | Your Java service calls it with an HTTP POST                               |
| **SDKs**     | Ready-made libraries so you don't hand-write HTTP calls       | `pip install anthropic`, then `client.messages.create(...)`                |
| **CLIs**     | Command line tools to use Claude from a terminal              | Run a quick prompt without writing a program                               |
| **Console**  | A website for the admin side of things                        | Create API keys, see how much you're spending, test prompts, deploy agents |

> **Think of it like a payments provider:** they give you an API, an SDK, and a dashboard. You write code against the API; you use the dashboard to check keys and usage. Same shape here.

### 3. The three layers

The lesson's main mental model: picture the platform as three layers stacked on top of each other.

```
┌─────────────────────────────────────────┐
│  CONTROLS      dashboards, evals        │  ← run it in production
├─────────────────────────────────────────┤
│  INFRASTRUCTURE  managed agents,        │  ← keep it running at scale
│                  retries, queues,       │
│                  observability          │
├─────────────────────────────────────────┤
│  PRIMITIVES    Messages API, tool use,  │  ← the pieces you call
│                files, web search, code  │
│                execution, MCP servers,  │
│                skills                   │
└─────────────────────────────────────────┘
```

#### Layer 1 — Primitives (the building blocks)

These are the actual things you call from your code.

- **Messages API** — send text, get text back. The main one.
- **Tool use** — let Claude call _your_ functions (e.g. `get_order_status(order_id)`).
- **Files** — give Claude a document to work with.
- **Web search** — let Claude look things up online.
- **Code execution** — let Claude run code to compute an answer.
- **MCP servers** — a standard way to plug external systems into Claude.
- **Skills** — packaged instructions for doing a specific kind of task well.

> **Example:** A refund bot uses the _Messages API_ to talk, _tool use_ to call your `issue_refund()` function, and _files_ to read the invoice PDF.

#### Layer 2 — Infrastructure (what keeps it alive at scale)

A prototype makes one API call. A real product makes thousands. This layer is the plumbing for that.

- **Managed agents** — Anthropic runs your agent for you, instead of you hosting it.
- **Retries** — automatically try again when a call fails.
- **Queues** — line up work so nothing is dropped during a traffic spike.
- **Observability** — logs and traces so you can see what happened and why.

> **Example:** Your demo worked fine for one ticket. On Monday morning 5,000 tickets arrive at once. Queues hold them, retries handle the failures, observability tells you which ones went wrong.

#### Layer 3 — Controls (the dials once it's live)

- **Dashboards** — usage, cost, traffic at a glance.
- **Evals** — tests that measure whether Claude's output is actually good, so you can tell if a change made things better or worse.

> **Example:** You change the system prompt. Evals tell you quality went up 8%; the dashboard tells you cost went up 3%. Now you can decide.

#### The shorthand to remember

> **Build with primitives → scale on infrastructure → run with control.**

The Claude Console is where the infrastructure and control layers live — it has sections for building, managing agents, and analytics.

### 4. Worked example: drafting help desk replies

**The task:** You run a help desk app. You want a button that drafts a reply to a ticket, in your team's tone.

**The flow — four steps:**

1. Define a client
2. Retrieve the ticket the chat refers to
3. Call `messages.create`
4. Return the response to the button to render

```python
client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-haiku-4-5",       # Haiku: a good fit for a simple drafting task
    max_tokens=1024,
    system=TONE_AND_GUIDELINES,
    messages=[
        {"role": "user", "content": ticket_content}
    ],
)

draft = response.content
```

### What each parameter does

| Parameter    | Job                                               | In this example                                                                |
| ------------ | ------------------------------------------------- | ------------------------------------------------------------------------------ |
| `model`      | Which model handles the request                   | Haiku — drafting a reply is a simple task, so you don't need the biggest model |
| `max_tokens` | Caps how long the response can be                 | 1024 — stops it writing an essay                                               |
| `system`     | The role Claude plays; your standing instructions | Your team's tone and reply guidelines                                          |
| `messages`   | An array of objects — the actual conversation     | One `user` message containing the ticket text                                  |

**Reading it as a sentence:**
_"Using Haiku, in at most 1024 tokens, acting as our support agent with our tone rules, here is the customer's ticket — write a draft reply."_

> **Why `system` and `messages` are separate:**
> `system` = the rules that never change ("always be polite, never promise a refund date").
> `messages` = the thing that changes every time (this particular ticket).
> Keeping them apart means you write the rules once and reuse them for every ticket.

### 5. The shift this represents

Notice what did **not** happen in that example: you didn't build a chatbot.

You took a product that already existed — a help desk with a UI and a button — and wired Claude into one feature of it. That's the real pattern.

> **From:** "ask Claude a question"
> **To:** "Claude is part of my product"

And when your product needs agents, the platform doesn't just hand you the model — with **managed agents**, it runs them for you.

### Quick glossary

- **API** — an address your code can send requests to and get answers back from.
- **SDK** — a library that wraps the API so you write normal function calls instead of raw HTTP.
- **Token** — a chunk of text (roughly a word-piece). Length and cost are measured in tokens.
- **System prompt** — standing instructions that define Claude's role and rules for every request.
- **Agent** — a program where Claude works in a loop, using tools, until a task is done — rather than answering once.
- **Eval** — an automated test that scores output quality, so you can compare versions.

---

## Your First API Call

### 1. The goal of this lesson

Saying "hello" to Claude is nice, but useless. This lesson sends Claude something **real** — a piece of buggy code — and gets **useful insight** back, in **under 20 lines of code**.

By the end you'll know the one function everything else in the platform builds on: `messages.create`.

### 2. Setup (3 steps)

#### Step 1 — Get an API key

Go to **platform.claude.com** which is also called claude console and create a key. You need to **purchase credits first** — the API is pay-as-you-go, it isn't included with a chat subscription.

#### Step 2 — Store the key safely

Put it in a **`.env.local`** file so it never enters version control.

```
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxx
```

> **Why this matters:**
> Hardcoding a key in a source file is _the_ classic way keys get leaked on GitHub. Once it's in a commit, it's in the history forever — even if you delete the line later. Bots scan public repos for keys within minutes.
>
> **Rule:** keys live in environment files; environment files live in `.gitignore`.

#### Step 3 — Install the SDK

```bash
npm install @anthropic-ai/sdk
```

> **Note:** you never pass the key in your code. `new Anthropic()` picks up `ANTHROPIC_API_KEY` from the environment automatically. That's the whole point of keeping it in `.env.local`.

### 3. The anatomy of a request

**Every API call goes through `messages.create`.** You specify three things:

| Thing            | What it means                                      | Simple example                                 |
| ---------------- | -------------------------------------------------- | ---------------------------------------------- |
| **`model`**      | Which Claude model handles the request             | `"claude-opus-4-7"`                            |
| **`max_tokens`** | A cap on how long the _response_ can be            | `1024`                                         |
| **`messages`**   | A list of objects with `user` or `assistant` roles | `[{ role: "user", content: "Hello, Claude" }]` |

The `messages` array is structured **just like a conversation** you'd have with Claude anywhere else — you say something (`user`), Claude replies (`assistant`), you say something back (`user`), and so on.

#### The most basic form

```javascript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const msg = await client.messages.create({
  model: "claude-opus-4-7",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: "Hello, Claude",
    },
  ],
});
```

### 4. A real example: reviewing buggy code

Now something more interesting than "hello." We point Claude at broken code and ask for a review.

```javascript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const buggyCode = `
function add(a, b) {
  return a - b;
}
`;

const response = await client.messages.create({
  model: "claude-opus-4-8",
  max_tokens: 1024,
  system:
    "You are a terse senior code reviewer. Give feedback in one paragraph.",
  messages: [{ role: "user", content: `Review this code:\n${buggyCode}` }],
});

for (const block of response.content) {
  if (block.type === "text") {
    console.log(block.text);
  }
}
```

**The bug:** a function called `add` that actually **subtracts**. Run it, and Claude spots exactly that and tells you in one paragraph.

### 5. Two things worth noticing

#### (a) The system prompt shapes the persona

```javascript
system: "You are a terse senior code reviewer. Give feedback in one paragraph.";
```

You don't configure a personality through settings or fine-tuning — **you just say what you want in plain English.**

> **Example of the difference:**
>
> | System prompt                                                             | What you get                                             |
> | ------------------------------------------------------------------------- | -------------------------------------------------------- |
> | _(none)_                                                                  | A friendly, thorough, multi-section review with headings |
> | `"You are a terse senior code reviewer. Give feedback in one paragraph."` | One tight paragraph, straight to the point               |
> | `"You are a patient mentor teaching a beginner."`                         | Gentle, explains _why_ the bug matters                   |
>
> Same code, same model — different instruction, different output.

Notice the system prompt was **not** in the basic example. It's optional, but it's the cheapest, highest-leverage thing you can add.

#### (b) The response content is an **array of blocks**, not a string

This is the part that trips people up. You cannot do `console.log(response.content)` and expect clean text.

```javascript
for (const block of response.content) {
  if (block.type === "text") {
    console.log(block.text);
  }
}
```

**Why an array?** Because Claude can return more than plain text in one response:

- `text` — normal written output
- `tool_use` — Claude asking to call one of your functions
- `thinking` — Claude's reasoning, when extended thinking is on

For a basic text reply there's usually just **one block of type `text`**. But because there _can_ be more, the safe habit is: **always loop, always check the type.**

### 6. From script to product

The same `messages.create` shape is the engine behind a real feature. Take a **"summarize" endpoint**:

```
1. Pull a meeting transcript out of the database
2. Hand it to Claude with a system prompt:
   "Extract insights and risks"
3. Save the result back on the row
4. Return it to the UI
```

**It's the same call — just wrapped in a route handler.**

---

## Choosing the Right Model

### 1. The problem in one sentence

You're shipping an app with Claude. **Which model do you pick?**

- Default to the **smartest** one → your API bill will surprise you.
- Pick the **cheapest** one → the output might not hold up.

Every model has different trade-offs, and the choice affects **both quality and cost**.

You choose with the **`model` parameter** in your API call.

### 2. The model tiers

Anthropic currently offers **four tiers**.

| Tier       | Speed                              | Cost                           | Best for                                                                       |
| ---------- | ---------------------------------- | ------------------------------ | ------------------------------------------------------------------------------ |
| **Fable**  | —                                  | Significantly higher than Opus | Your toughest challenges — only where the extra capability is worth paying for |
| **Opus**   | Slowest of the three core families | Highest of the three           | Deep reasoning, complex analysis, multi-step coding, nuanced writing           |
| **Sonnet** | Balanced                           | Balanced                       | **Most production work** — the sweet spot                                      |
| **Haiku**  | Fastest                            | Lowest                         | High-volume, low-complexity work: classification, extraction, routing          |

#### The three core families, in plain words

**Claude Opus** — the most capable of the three core model families, but also the slowest and highest cost.

> _Use it for:_ reviewing a legal contract, refactoring across many files, writing something where the phrasing genuinely matters.

**Claude Sonnet** — a balanced combination of intelligence, speed, and cost. Works well for most production work.

> _Use it for:_ drafting a client update, summarizing a meeting, answering support questions with judgment involved.

**Claude Haiku** — fastest and lowest cost, optimized for speed and cost efficiency rather than maximum intelligence.

> _Use it for:_ "is this email spam or not?", pulling the invoice number out of a PDF, deciding which team a ticket belongs to.

### 4. Comparing the tiers side by side

Don't just talk about the difference — **measure it**. Send the same prompt through all three models and watch latency and token counts.

```python
models = ["claude-haiku-4-5", "claude-sonnet-4-6", "claude-opus-4-7"]

for model in models:
    response = client.messages.create(
        model=model,
        max_tokens=300,
        messages=[{"role": "user", "content": prompt}],
    )
    print(model, response.usage)
```

#### Two things going on here

**1. The loop swaps only the `model` field.**
Same prompt, same `max_tokens`. Only the model changes — so any difference you see is caused by the model and nothing else. That's what makes it a fair comparison.

**2. `response.usage` gives you input and output tokens straight back from the API.**
**This is what your bill is calculated on.** So you're not guessing at cost — you're reading it directly.

### 5. Routing different work to different models

In a real app, you **don't pick one model for everything**. You route different kinds of work to different models **inside the same endpoint**.

**Example — an operations dashboard with a document processing route:**

```
        incoming file
             │
             ▼
    ┌─────────────────┐
    │ classify: HAIKU │   every file gets classified — cheap, fast, high volume
    └────────┬────────┘
             │
    ┌────────┴────────┬──────────────────┐
    ▼                 ▼                  ▼
 client update    RFP response       (other)
   SONNET            OPUS
 draft the reply  needs real reasoning
```

- Every incoming file gets **classified with Haiku**.
- Client updates get **drafted with Sonnet**.
- Only RFP responses **reach for Opus**.

> **One queue, three models, picked per task.**

This is where the savings really come from: the cheap model handles the step that runs on _every_ item, and the expensive model only touches the small slice that needs it.

---

---

# Module 2: Teaching your Agent

## The Agent Loop Explained

### 1. Why we need more than one API call

So far every call has been: **you ask → Claude answers → done.** One question, one response.

But to **automate a workflow**, that isn't enough. Claude needs to:

> **act → look at the result → decide what's next → keep going**

That repeating pattern is what people mean by **"agentic workflows."**

> **Simple example:** Ask "what should I wear today?" — Claude can't answer, because it doesn't know the weather. It needs to _go find out first_, then answer. One call can't do that. Two can.

### 2. What an agent actually is

> **An agent is an autonomous version of Claude, running both sides of the messaging loop without a human in the middle.**

Normally _you_ are the one replying to Claude. In an agent, **your code** replies to Claude — automatically — with whatever the tool produced.

An agent **receives a task, picks a tool, and executes code in a loop until Claude decides the task is done.**

#### The five steps

1. **Send** a message to Claude with tools available.
2. **Claude responds** with either a final answer **or** a request to use a tool you defined.
3. **Your code executes** that tool.
4. **You send the result back** to Claude.
5. **Repeat** until the stop reason is `end_turn`.

```
        ┌──────────────────────────────────────┐
        │                                      │
        ▼                                      │
   send messages ──▶ Claude replies            │
                          │                    │
              ┌───────────┴───────────┐        │
              ▼                       ▼        │
      stop_reason =            stop_reason =   │
        "tool_use"              "end_turn"     │
              │                       │        │
      run the tool,                 DONE       │
      append the result ─────────────────────┘
      back into messages
```

> **Think of it as a conversation where the turns alternate:** the user kicks things off, the agent calls a tool, the tool returns a result, and the agent keeps going until it has an answer.

### 3. A minimal working example

To see the loop run end to end without dragging in a database or a UI, we wire up a **fake tool called `get_weather`** and ask _"What should I wear in Austin today?"_

Claude has **no way to know the weather on its own**, so it _has to_ call the tool, read the result, and only then give an answer. That's what forces the loop to actually run.

```python
import anthropic

client = anthropic.Anthropic()

# The tools array tells Claude what's available:
# a name, a description, and a JSON schema for the inputs.
tools = [
    {
        "name": "get_weather",
        "description": "Get the current weather for a city.",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "The city to get weather for",
                }
            },
            "required": ["city"],
        },
    }
]

# run_tool is just a hardcoded lookup.
# In a real app, this would hit your database, an API, whatever.
def run_tool(name, tool_input):
    if name == "get_weather":
        return f"Weather in {tool_input['city']}: 95F, sunny"
    raise ValueError(f"Unknown tool: {name}")

messages = [
    {"role": "user", "content": "What should I wear in Austin today?"}
]

# The agent loop. Each iteration sends messages to Claude
# and switches on the response's stop reason.
while True:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        tools=tools,
        messages=messages,
    )

    if response.stop_reason == "end_turn":
        # Claude is done. Print the final text and break.
        for block in response.content:
            if block.type == "text":
                print(block.text)
        break

    if response.stop_reason == "tool_use":
        # Find the tool use blocks in the response and run each one.
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                result = run_tool(block.name, block.input)
                tool_results.append(
                    {
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result,
                    }
                )

        # Push the assistant's response and our tool results
        # back into messages, then loop again so Claude can answer.
        messages.append({"role": "assistant", "content": response.content})
        messages.append({"role": "user", "content": tool_results})
```

### 4. Three pieces to notice

#### (a) The `tools` array — describing what's available

Three parts per tool:

| Part               | What it does                                                                   |
| ------------------ | ------------------------------------------------------------------------------ |
| **`name`**         | What Claude calls it — `get_weather`                                           |
| **`description`**  | Plain English: _when_ to use it. This is how Claude decides                    |
| **`input_schema`** | A JSON schema for the inputs — what arguments it takes, and which are required |

> **Important:** you are not giving Claude the _code_. You're giving it a **menu** — the names of things it can ask for and what information each one needs. Claude never runs anything itself; it only ever _requests_.
>
> The `description` is doing real work here. "Get the current weather for a city" is what tells Claude that a question about what to wear in Austin needs this tool.

#### (b) `run_tool` — where your actual work happens

```python
def run_tool(name, tool_input):
    if name == "get_weather":
        return f"Weather in {tool_input['city']}: 95F, sunny"
```

Here it's **just a hardcoded lookup**. In a real app **this would hit your database, an API, whatever** — a SQL query, a REST call, a file read, a payment API.

> This is the line between the two worlds: **Claude decides _which_ tool and _with what arguments_. Your code decides _what actually happens._** Nothing runs that you didn't write.

#### (c) The loop — switching on `stop_reason`

Each iteration sends the messages to Claude and **switches on the response's stop reason**:

**On `end_turn`** → Claude is done. Print the final text and `break`.

**On `tool_use`** → find the `tool_use` blocks, run each one, then push **two** things back into `messages`:

```python
messages.append({"role": "assistant", "content": response.content})  # what Claude asked for
messages.append({"role": "user", "content": tool_results})           # what the tool returned
```

…and loop again so Claude can answer.

> **Why append both?** The conversation must stay complete and in order. Claude needs to see its own request _and_ the answer to it — otherwise the tool result appears out of nowhere with nothing to attach to.
>
> **Why `tool_use_id`?** It ties each result back to the exact request that asked for it. If Claude requested three tools at once, this is how each answer finds its question.
>
> **Note the roles:** the tool result is sent with `role: "user"`. From the API's point of view, your code is now playing the user's part — which is exactly what "running both sides of the loop" means.

### 5. Running it — what you actually see

**Two turns:**

| Turn  | `stop_reason` | What happens                                                                               |
| ----- | ------------- | ------------------------------------------------------------------------------------------ |
| **1** | `tool_use`    | Claude requests `get_weather` for Austin. Your code returns the temperature and conditions |
| **2** | `end_turn`    | Claude tells you to wear something light and breathable                                    |

**Two API calls, one tool execution, one final answer. That's the entire loop.**

> **Everything you build with the Claude API is going to be similar to this.**

### 6. The same loop in production

In a real environment, this exact loop powers something like an **auto-review endpoint**:

> A **compliance agent** reads a structural report, looks up the relevant building codes via a tool, and writes risk findings back to the database one by one as it works.

**The shape of the loop is identical to what you just ran.** The differences are only:

- **Real tools** instead of a mock weather lookup
- **Results stream back to the UI** as server-sent events
- **Findings get persisted** to a risk-finding table

---

## What is Tool Use?

### 1. The problem tools solve

Your workflows depend on lots of things Claude can't see: **project management software, databases, files.**

Claude **can't just check these things itself.** It relies on **tools**, which give Claude access to **external data and actions**.

> **Simple example:** Ask Claude "how many open tickets does Priya have?" — it has no idea. Your Jira board isn't in its head. Give it a `get_open_tickets(assignee)` tool, and now it can find out.

### 2. What a tool actually is

> **A tool is a function you define and expose to Claude. You describe what it does and what inputs it takes, and Claude decides when to call it.**

### The key thing to remember

> ### Claude doesn't execute the tool — **your code does.**

```
1. Claude REQUESTS a tool call
        ↓
2. YOUR CODE executes the function
        ↓
3. The RESULT goes back to Claude, and it keeps going
```

### 3. How tools are defined

Tools are **JSON schemas with three parts**, passed in the request body as a **`tools` array**.

| Part               | Purpose                                                           |
| ------------------ | ----------------------------------------------------------------- |
| **`name`**         | The identifier Claude uses to request it                          |
| **`description`**  | What Claude **reads to decide whether to call it**                |
| **`input_schema`** | The shape of the inputs — types, descriptions, which are required |

#### ⚠️ The description is the most important line you write

> **If you write a vague description, you get bad tool use. This is the number one reason agents misfire or don't grab the tools that are available to them. Be specific.**

#### A real tool definition

```json
{
  "name": "lookup_building_code",
  "description": "Look up a specific building code section by its identifier. Returns the full text of that code section.",
  "input_schema": {
    "type": "object",
    "properties": {
      "section": {
        "type": "string",
        "description": "The building code section to look up"
      }
    },
    "required": ["section"]
  }
}
```

Note the description does **two** jobs: it says what the tool _does_ **and** what it _returns_. Both help Claude decide.

### 4. What happens when it's used

Say we send an agent a compliance report.

**Turn 1:** Claude comes back with **`stop_reason: "tool_use"`** — **that's our signal.**

**Then our loop:**

1. Calls `lookup_building_code` with the parameter Claude requested
2. Feeds the result back as a **tool result** — a **`user` message containing a `tool_result` block tied to the tool call's `id`**

**And Claude keeps going.** From there, you keep calling tools and returning results **until it has what it needs.**

> **The `id` link:** every tool request Claude makes carries an id, and your result must carry the same `tool_use_id`. That's the thread tying the answer to the question — essential when several tools are requested at once.

### 5. Multiple tools: letting Claude pick

One tool is useful. **The interesting part is giving Claude multiple tools and watching it pick which one to use, in what order.**

**Scenario:** you're packing for a **three-day trip to Denver**. You want **today's weather** _and_ **the forecast for the next few days**. So declare **two** tools.

```javascript
const tools = [
  {
    name: "get_weather",
    description: "Get today's current weather for a city.",
    input_schema: {
      type: "object",
      properties: {
        city: { type: "string", description: "The city to check" },
      },
      required: ["city"],
    },
  },
  {
    name: "get_forecast",
    description: "Get the weather forecast for the next few days for a city.",
    input_schema: {
      type: "object",
      properties: {
        city: { type: "string", description: "The city to check" },
      },
      required: ["city"],
    },
  },
];
```

Notice the two descriptions are deliberately distinguishable: **"today's current weather"** vs **"the next few days."** That single difference is what lets Claude choose correctly.

#### The loop is identical to before

**The only new piece** is a `runTool` function that **dispatches on the tool name with a switch statement** — this block is just **where your code actually runs**.

```javascript
function runTool(name, input) {
  switch (name) {
    case "get_weather":
      return getWeather(input.city);
    case "get_forecast":
      return getForecast(input.city);
  }
}

while (true) {
  const response = await client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 1024,
    messages,
    tools,
  });

  if (response.stop_reason !== "tool_use") {
    // Claude is done — this is the final answer
    break;
  }

  messages.push({ role: "assistant", content: response.content });

  const toolResults = response.content
    .filter((block) => block.type === "tool_use")
    .map((block) => ({
      type: "tool_result",
      tool_use_id: block.id,
      content: runTool(block.name, block.input),
    }));

  messages.push({ role: "user", content: toolResults });
}
```

> **`.filter(...).map(...)`** — filter picks out only the `tool_use` blocks (the response may also contain text), and map turns each one into a matching `tool_result`. **Plural, because Claude can request several tools in a single turn.**

#### Adding a third tool

> **Want a third tool? Add it to the array, add a case to the switch, and you're done.**

#### What you see when you run it

Claude calls **`get_weather`** and then **`get_forecast`** — **sometimes in the same turn, sometimes one after the other.** Then it answers: _pack layers, expect snow flurries today, warming through the week._

#### How Claude chose

> **It read the descriptions, mapped your prompt to "today's weather" and "the next few days," and picked the right tool for each.**
>
> **That's why your tool descriptions really matter.** They're not documentation for humans — they're the actual decision logic.

### 6. The tool runner: skip the boilerplate

Two red flags with the code above:

1. **That's a lot of code for two simple lookups.**
2. **In a real codebase, you don't want to handwrite JSON schemas for every function you have. It's like writing your code twice.**

#### What the tool runner does

It ships in the Claude SDK for **TypeScript, Python, and Ruby**. The runner:

- **Takes your actual functions**
- **Reads the types and docs to build the schema for you**
- **Handles the entire tool use / tool result loop internally**

Your code shrinks to: **describe the tool, send the prompt, wait for the result.**

```typescript
// The same two lookups we ran by hand — just plain TypeScript functions
function getWeather(city: string) {
  // ...existing lookup
}

function getForecast(city: string) {
  // ...existing lookup
}

const runner = client.beta.messages.toolRunner({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content:
        "I'm packing for a three-day trip to Denver. What's the weather today and over the next few days?",
    },
  ],
  tools: [getWeather, getForecast],
});

// Returns the final assistant message after all the tool ping-pong has settled
const finalMessage = await runner.untilDone();
```

#### Same scenario, a fraction of the code

- **No `while` loop, no stop reason switch, no manually pushing tool results back into messages** — the runner handles all of that.
- **No JSON schemas**, so you don't write things twice.
- The two functions are **the same lookups we ran by hand a minute ago, just plain TypeScript.**
- **`runner.untilDone()`** returns the final assistant message **once everything has settled.**

**Run it, and you get the same answer.**

### Manual loop vs tool runner

|                        | Manual loop    | Tool runner               |
| ---------------------- | -------------- | ------------------------- |
| JSON schemas           | You write them | Built from your functions |
| The loop               | You write it   | Handled internally        |
| Stop-reason switch     | You write it   | Handled internally        |
| Pushing results back   | You write it   | Handled internally        |
| Control over each step | Full           | Less — it's abstracted    |

> Both are valid. Write the loop yourself when you need to inspect, log, or intervene between turns; use the runner when you just want the answer.

### 7. Real tools wrap your existing code

**In real life, your tools wouldn't be hardcoded weather data. They'd wrap actual functions you already have in your application.**

**Example — a compliance review agent:**

> Its tools are **thin wrappers around `lookup_building_code` and `search_building_code` functions that already exist in the codebase.** With the tool runner, **you pass those functions in directly**, and the agent **cites specific code sections in every finding it writes** — **no schema writing required.**

> **The takeaway:** you are usually not writing new capability for the agent. You're **exposing capability you already have.** The tool layer is a thin door onto existing code.

---

## What is Thinking? — Notes

### 1. The failure mode we're trying to avoid

Some tasks need **more than a quick answer.**

> **Ask a model a multi-step question and have it answer immediately, and it can confidently get it wrong.**

The dangerous word there is **confidently**. It doesn't say "I'm not sure" — it gives you a clean, well-written answer that happens to be incorrect.

**Extended thinking** is the feature that gives Claude that paper.

### 2. What extended thinking is

> **Extended thinking lets Claude reason step by step before producing a final response.**

When it's enabled:

1. Claude generates **internal reasoning tokens** — often called a **chain of thought**
2. Then it delivers the answer

#### The reasoning isn't hidden

> **You can see it in the response alongside the final text.**

This is worth pausing on. You get **thinking blocks** _and_ **text blocks** back. That means you can:

- Check _how_ Claude reached a conclusion, not just what it concluded
- Debug a wrong answer by seeing where the reasoning went sideways
- Show the working to a reviewer who needs to trust the output

> **This is why the response `content` is an array of blocks** (from Lesson 2). Thinking is one of those block types.

### 3. Adaptive thinking on Opus 4.7

> **With Opus 4.7, thinking is adaptive. You don't pick a token budget. You just turn it on, and Claude decides dynamically when to think and how much.**

```python
thinking={"type": "adaptive"}
```

That's the whole switch. No budget maths, no guessing how many reasoning tokens the problem deserves. Instead of manually setting a thinking token budget, adaptive thinking lets Claude dynamically determine when and how much to use extended thinking based on the complexity of each request. What you still control is a coarse dial, output_config.effort (low / medium / high), which shapes overall spend rather than setting an exact token count.

#### Controlling how much it thinks — the `effort` parameter

| Level    |                   |
| -------- | ----------------- |
| `low`    |                   |
| `medium` |                   |
| `high`   | **← the default** |
| `xhigh`  | extra high        |
| `max`    |                   |

```python
output_config={"effort": "high"}
```

#### ⚠️ One gotcha

> **`effort` goes inside `output_config`, NOT next to the `thinking` block.**

```python
# ✅ Correct
thinking={"type": "adaptive"},
output_config={"effort": "high"},

# ❌ Wrong
thinking={"type": "adaptive", "effort": "high"},
```

Easy mistake to make, because they feel like they belong together conceptually. They don't, structurally.

### 4. When to use it — and when to skip it

### ✅ Extended thinking helps with

| Use case                                          | Why                                                              |
| ------------------------------------------------- | ---------------------------------------------------------------- |
| **Math and multi-step logic**                     | Each step depends on the last; skipping ahead breaks it          |
| **Code debugging**                                | You have to trace the flow to find where it actually goes wrong  |
| **Regulatory analysis**                           | Rules interact; a clause in one place changes what another means |
| **Anything with trade-offs or comparing options** | You must hold several options side by side before choosing       |

#### ❌ Skip it for

- **Simple classification**
- **Extraction**
- **Boilerplate**

> **For those tasks it just adds latency and cost without actually improving the results.**

> **The rule in one line:** if _you_ would need to stop and work it out, turn thinking on. If you'd answer without pausing, leave it off.

### 5. Thinking in action

**The setup:** an agent loop with **one weather tool**. We ask Claude to **plan a road trip out of San Francisco — two stops, weighing weather and drive time.**

> **That's a real trade-off** — the kind of question **where thinking earns its keep.** A nice stop that's too far, or a close stop with bad weather, are both wrong; you have to weigh them against each other.

```python
import anthropic

client = anthropic.Anthropic()

weather_tool = {
    "name": "get_weather",
    "description": "Get the current weather for a city.",
    "input_schema": {
        "type": "object",
        "properties": {
            "city": {"type": "string", "description": "City name"}
        },
        "required": ["city"],
    },
}

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},   # low | medium | high | xhigh | max
    tools=[weather_tool],
    messages=[
        {
            "role": "user",
            "content": "Plan a road trip out of San Francisco with two stops, "
                       "weighing weather and drive time.",
        }
    ],
)
```

> **Note `max_tokens=16000`.** Much larger than the 1024 we've used before — reasoning tokens count against your budget too, so a thinking call needs headroom.

#### What you get back

**The output is more interesting than usual.** You'll see:

```
  ┌──────────────────┐
  │  thinking block  │  Claude works through the trade-offs
  └──────────────────┘
           ↓
  ┌──────────────────┐
  │   tool calls     │  checks the weather in each candidate city
  └──────────────────┘
           ↓
  ┌──────────────────┐
  │   text block     │  the actual recommendation
  └──────────────────┘
```

> **The reasoning is visible — that's the whole point.**

**Notice thinking and tools combine.** Claude reasons about _which_ cities are worth checking, calls the tool to find out, and reasons again about what came back. They're not alternatives; they work together.

### 6. Why this matters in production

> **In a production app, this is the difference between an agent that finds problems one at a time and an agent that connects them.**

---

---

# Module 3: Extending your Agent

## Built-in Tools

### 1. The idea in one line

You can build your own custom tools (Lesson 5) — but **some capabilities are common enough that Anthropic ships them pre-built.**

> **You don't write the code. You don't host the sandbox. You just declare the tool, and Anthropic runs it.**

### 2. Server tools: declared by you, run by Anthropic

**Anthropic provides server tools that run on their infrastructure.** You don't execute these — **Anthropic does.**

#### The consequence: no agent loop needed

> **Claude calls the tools on its own, and the result comes back inside the same response.**

Compare with what you built in Lessons 4 and 5:

```
   CUSTOM TOOL (you run it)          SERVER TOOL (Anthropic runs it)
   ─────────────────────────         ───────────────────────────────
   send messages + tools             send messages + tools
        ↓                                 ↓
   stop_reason: "tool_use"           ...Anthropic does the work...
        ↓                                 ↓
   YOU run the function              response ALREADY has the result
        ↓
   YOU push the result back
        ↓
   loop again
        ↓
   stop_reason: "end_turn"
```

**One round trip instead of several. No `while` loop, no `stop_reason` switch, no pushing results back.**

#### The main server tools

| Tool               | What it does                                                 |
| ------------------ | ------------------------------------------------------------ |
| **Web search**     | Searches the internet and **returns results with citations** |
| **Code execution** | **Writes and runs Python in a sandbox**                      |
| **Web fetch**      | **Retrieves full content from URLs**                         |

> **Search vs fetch:** search is "go find pages about X." Fetch is "go read _this specific_ page." Different jobs.

### 3. Two server tools in one file

Two `messages.create` calls — one with **web search**, one with **code execution**.

```python
import anthropic

client = anthropic.Anthropic()

# Call 1: web search — Anthropic runs the search server-side
search_response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    tools=[{"type": "web_search_20260209", "name": "web_search"}],
    messages=[
        {"role": "user", "content": "What is Anthropic's latest model release? Answer in one sentence."}
    ],
)

for block in search_response.content:
    if block.type == "server_tool_use":
        print(f"Tool call: {block.name} — {block.input}")
    elif block.type == "text":
        print(block.text)

# Call 2: code execution — Claude writes and runs Python in a sandbox
code_response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    tools=[{"type": "code_execution_20260120", "name": "code_execution"}],
    messages=[
        {"role": "user", "content": "Calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]"}
    ],
)

for block in code_response.content:
    if block.type == "server_tool_use":
        print(f"Tool call: {block.name} — {block.input}")
    elif block.type == "bash_code_execution_tool_result":
        print(f"stdout: {block.content.stdout}")
    elif block.type == "text":
        print(block.text)
```

> **Note the tool declaration shape.** A custom tool needed a `name`, `description`, and full `input_schema`. A server tool needs only a **`type`** (with a dated version, e.g. `web_search_20260209`) and a **`name`**. **No description, no schema — Anthropic already knows what the tool does.**
>
> **Why the date in the type?** It pins the version, so a future change to the tool doesn't silently change your app's behaviour.

### 4. Two things to notice

#### (a) There's no agent loop here

> **We don't switch on `stop_reason`. We don't push tool results back. Anthropic runs the tool server-side, and the response already contains the result.**

#### (b) The response has new block types

| Block type                                                         | What it holds                                       |
| ------------------------------------------------------------------ | --------------------------------------------------- |
| **`server_tool_use`**                                              | The tool call — the name and the inputs Claude used |
| **Code execution tool result** (`bash_code_execution_tool_result`) | The output, including `stdout`                      |
| **`text`**                                                         | The regular final answer                            |

### 5. Running it

**Web search** → you'll see:

1. Claude's tool call printed
2. A one-sentence answer about the latest model release, **with the search citations folded in**

**Code execution** → you'll see:

1. **The actual Python Claude wrote**
2. **The `stdout` from the sandbox running it**
3. A final text answer

#### The point

> **We didn't have to spin up a search crawler. We didn't run a Python sandbox. We declared two tools and got both for free.**

### 6. The other category: client tools

Worth knowing this category exists.

> **Client tools run where your code runs. They're shipped in the Claude SDK, so you don't have to define the schema yourself.**

| Client tool | What it does                                               |
| ----------- | ---------------------------------------------------------- |
| **Memory**  | Claude **reads and writes memory across sessions**         |
| **Bash**    | **A persistent bash shell** so Claude can execute commands |

> **They have the same shape as a custom tool, but the SDK gives you the schema and a sensible runner.**

#### The three categories side by side

|                            | **Custom tools**       | **Client tools** | **Server tools**                      |
| -------------------------- | ---------------------- | ---------------- | ------------------------------------- |
| **Who writes the schema?** | You                    | The SDK          | Not needed                            |
| **Who writes the code?**   | You                    | The SDK          | Anthropic                             |
| **Where does it run?**     | Your machine           | Your machine     | Anthropic's infra                     |
| **Agent loop needed?**     | Yes                    | Yes (SDK helps)  | **No**                                |
| **Examples**               | `lookup_building_code` | memory, bash     | web search, code execution, web fetch |

### 7. Why this matters in production

> **This is the shortest path to features that would otherwise take weeks.**

**Example:** **web search can power a fact-check endpoint** that **verifies every numeric and regulatory claim in a draft against the live web.**

#### ⚠️ One important reminder

> **Just because something is validated on the internet doesn't mean it's true. Always double-check Claude's work.**

Web search removes the _infrastructure_ problem, not the _truth_ problem. The citations tell you where a claim came from — they don't tell you the source was right. **For anything consequential, a human still reviews it.**

---

## Skills

### 1. What a Skill is

> **Skills are folders of instructions, scripts, and resources that Claude loads dynamically to improve performance on specialized tasks.**

At the core of every Skill is a **`SKILL.md`** file — **a packaged set of instructions you upload once and then attach to any `messages.create` call.**

#### What you're really doing

> **You're teaching Claude how _you_ do something:** your status report format, your review checklist, your release notes.
>
> **Claude reads the Skill, follows the procedure, and produces output in your shape.**

> **Everyday example:** A new person joins your team. You don't explain the release-notes format from scratch every time they write one — you hand them the team's written guide once, and they follow it. A Skill is that guide, for Claude.

### 2. Skills vs. Tools — the key distinction

The two solve **different problems**:

|                  | **Tools**                                          | **Skills**                                                                                          |
| ---------------- | -------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Purpose**      | Connect Claude to **data and actions**             | Teach Claude **a procedure**                                                                        |
| **Example**      | _"Look up this code section," "send this email"_   | _"Generate the daily status report following this template"_                                        |
| **What happens** | Claude calls the tool, and **something else runs** | It's a **playbook Claude reads and follows** — which sometimes means running bundled scripts itself |

#### The one-line way to remember it

> ### **Tools are about _what_ Claude can do.**
>
> ### **Skills are about _how_ you want it done.**

> **They're complements, not alternatives.** A status report Skill might _use_ a tool to fetch the activity log. The tool gets the data; the Skill decides what the report looks like.

### 3. Skills load progressively

**One more thing worth knowing:**

> **Skills don't load fully into context on startup. Only the name and description load at first. When your agent decides a Skill is relevant, it then loads the full Skill into context.**

```
  At startup:                    When needed:
  ───────────                    ────────────
  Skill A: name + description    Skill B's FULL content
  Skill B: name + description  ──▶  loads into context
  Skill C: name + description       (A and C stay as
  Skill D: name + description        headlines only)
  ...

  cheap — just headlines         only the one that's relevant
```

> **Why it matters: that keeps your context lean even when many Skills are available.** You can have twenty Skills attached without paying for twenty Skills' worth of tokens on every call.

> **Note the echo of Lesson 5:** just like a tool, the **description is what drives the decision**. A Skill with a vague description won't get loaded when it should. Write it specifically.

### 4. Uploading a Skill

> **Skills are uploaded once to your workspace, then referenced by ID.**

You can upload **directly on the Claude Platform**, or do it **programmatically**:

```python
skill = client.beta.skills.create(
    display_title="Status Report Generator",
    files=files_from_dir("status-report-skill"),  # folder containing SKILL.md
)

print(skill.id)  # reference this ID in future requests
```

**Upload once → reuse forever by ID.** You're not re-sending the instructions on every request.

#### The example: a status report generator

> **All the rules for what makes a good status report — sections, tone, how to summarize, how to handle blockers — live in a Skill packaged ahead of time.**
>
> **The activity log itself is just a string passed in at request time.**

**This is the split worth internalizing:**

| Lives in the Skill (fixed) | Passed in the request (changes) |
| -------------------------- | ------------------------------- |
| Sections and their order   | Today's activity log            |
| Tone                       |                                 |
| How to summarize           |                                 |
| How to handle blockers     |                                 |

### 5. Attaching a Skill to a request

> **Skills attach to a request through the container configuration — a `skills` array inside the `container`, where each entry names a `skill_id` and `version`.**

```python
response = client.beta.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=4096,
    betas=["skills-2025-10-02", "code-execution-2025-08-25"],
    container={
        "skills": [
            {
                "type": "custom",
                "skill_id": skill.id,
                "version": "latest",
            }
        ]
    },
    tools=[
        {
            "type": "code_execution_20250825",
            "name": "code_execution",
        }
    ],
    messages=[
        {
            "role": "user",
            "content": f"Generate the daily status report from this activity log:\n\n{activity_log}",
        }
    ],
)
```

#### Three things worth pointing out

**(a) It's the beta client**

> We're calling **`client.beta.messages.create`**, not the standard one, and **passing the skills feature via the beta header**. **As of this video, Skills are still a beta feature.**

**(b) `container.skills` is where the Skill attaches**

> **It's a list, so you can layer multiple Skills onto one call.**
>
> _Example:_ a "house writing style" Skill **plus** a "status report format" Skill on the same request.

**(c) Code execution is turned on too**

> **Skills often pair well with code execution, because Skill procedures can do real work — like running scripts in a terminal.**
>
> Remember Skills are _folders_ — instructions **plus scripts and resources**. Code execution is what lets those bundled scripts actually run.

### 6. Running it

> **The output is a status report formatted exactly the way the Skill says to format it. Sections, tone, blocker handling — all of it comes from the `SKILL.md` file you uploaded.**

#### The line that captures the whole idea

> ### **The user prompt is one line; the procedure lives in the Skill.**

```
  WITHOUT a Skill                      WITH a Skill
  ──────────────                       ────────────
  A 600-word prompt with the           "Generate the daily status
  full template, tone rules,            report from this activity
  section order, blocker                log: {log}"
  handling — pasted in
  every single time                     (the rest lives in SKILL.md)
```

#### In production

> **This is how a team standardizes output across an entire feature.** With a daily status report endpoint, **every PM gets the same structure, the same tone, the same sections, in the same order — without anyone copy-pasting a template into a prompt.**

> **The maintenance win:** when the format changes, you update `SKILL.md` in one place. Nobody has to hunt down a template pasted across a dozen prompts.

---

## MCP

### 1. The question this lesson answers

We already have **tools, skills, and connectors.** So **why does MCP exist?**

> #### The answer comes down to **who maintains the integration code.**

### 2. The maintenance problem

**The scenario:** your agent needs to **pull tasks from Jira, check a Google Calendar, and search Slack** — all in one go.

**With custom tools:**

> You have to write **three integrations.** **That part is doable.**
>
> **The painful part comes after:** you also have to **maintain those integrations every time one of those services changes its API — which happens often.**
>
> **Congratulations, you're now maintaining a pile of third-party API wrappers.**

> **Think about what that actually means:** none of that work is _your product_. You didn't set out to build an JIRA client. But every time JIRA ships a change, someone on your team stops what they're doing and fixes your wrapper.

#### What MCP changes

> **MCP shifts that maintenance to the service provider.**

- **JIRA publishes an MCP server.**
- **Slack publishes one.**
- **Google publishes one.**

**Each server exposes its own tools — with descriptions, schemas, and authentication — through a standard protocol.**

> #### **When their API changes, they update their server. You change nothing.**

> **The word "standard protocol" is doing the heavy lifting.** Because every server speaks the same protocol, Claude can talk to _any_ of them without you writing per-service code. That's the whole point of a standard.

### 3. Tools vs. Skills vs. MCP

**These three features do different jobs:**

|                         | **Tools**                                        | **Skills**                             | **MCP**                           |
| ----------------------- | ------------------------------------------------ | -------------------------------------- | --------------------------------- |
| **Connects/teaches**    | Claude → **your internal systems**               | Claude → **a procedure**               | Claude → **third-party services** |
| **Examples**            | Your database, project tracker, proprietary APIs | Your report template, review checklist | Asana, Slack, Linear, Google      |
| **Who writes the code** | **You**                                          | You (as instructions)                  | **The service provider**          |
| **Who maintains it**    | **You own the code, so you own the maintenance** | You                                    | **They do**                       |

#### The short version

> #### **Tools are for your stuff.**
>
> #### **Skills are for your processes.**
>
> #### **MCP is for everyone else's stuff.**

### 4. Connecting to an MCP server

**The cleanest way to get a feel for MCP:** point Claude at any MCP server and **let it discover what's there.**

Example uses the **JIRA MCP server**, with connection details and auth token in a **`.env` file**.

#### Two pieces work together in the request

| Key                              | Job                                                                                                                          |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **`mcp_servers`**                | **Declares the connection** — a `type`, a `url`, a `name` to refer to it by, and optionally an auth token                    |
| **A tool of type `mcp_toolset`** | **Configures which tools Claude can use from that server.** **The default is all of them** — this is where you scope it down |

```python
import os
import anthropic

client = anthropic.Anthropic()

response = client.beta.messages.create(
    model="claude-opus-4-8",
    max_tokens=1000,
    messages=[
        {"role": "user", "content": "What tools do you have available?"}
    ],
    mcp_servers=[
        {
            "type": "url",
            "url": "https://mcp.jira.app/mcp",
            "name": "jira",
            "authorization_token": os.environ["JIRA_MCP_TOKEN"],
        }
    ],
    tools=[
        {
            "type": "mcp_toolset",
            "mcp_server_name": "jira",
        }
    ],
    betas=["mcp-client-2025-11-20"],
)

print(response)
```

> **The `name` ties the two halves together:** you name the server `"jira"` in `mcp_servers`, then refer back to it with `mcp_server_name: "jira"` in `tools`.

#### The thing to notice

> **We never wrote a single tool schema.**
>
> **Claude introspects the server, gets the list of tools and their schemas back, and picks the right one for the prompt.**

> **⚠️ Beta:** as of this lesson, the MCP connector is **in beta** — note the **beta header** (`betas=["mcp-client-2025-11-20"]`) in the request.

#### Running it

**If your MCP URL points at JIRA's MCP endpoint, Claude lists JIRA's tools and then calls one.** **The same works for basically any compliant server.**

> **We didn't define a single tool. We didn't write a JIRA client. JIRA is maintaining that.**

### 5. Filtering which tools Claude can use

> **MCP servers often expose many, many tools — and you don't always want Claude using all of them.**

**Two reasons to scope down:**

1. **Maybe you don't want it to have write permissions**
2. **Maybe you just don't want all those tool definitions taking up context**

#### The fix: deny by default, allow specifically

> **Disable everything by default, then enable only the specific tools you want.**

```python
tools=[
    {
        "type": "mcp_toolset",
        "mcp_server_name": "slack",
        "default_config": {
            "enabled": False,
        },
        "configs": {
            "search_messages": {"enabled": True},
            "list_channels": {"enabled": True},
        },
    }
]
```

**Result:**

> **Now Claude can search Slack and list channels, but it can't post or delete.**

> **Why this matters:** **it's useful when you trust a service for reads but don't want Claude writing on your behalf by accident.**
>
> **Note the pattern is deny-by-default, not allow-by-default.** If the server adds a new tool tomorrow, it arrives **disabled**. You never get surprised by a capability you didn't ask for. That's the safer default, and it's worth using even when you think you want everything.

---

## Context Management

### 1. The problem

**Every request you send Claude has a context window.**

> **A million tokens sounds like a lot, but it runs out faster than you think once you're shipping a real agent.**

**Context management is how you stay inside the window without losing what matters.**

> **Why it runs out fast:** an agent loop doesn't send one message — it re-sends the _whole conversation_ on every turn. Turn 10 carries turns 1–9 with it, plus every tool result along the way. Growth is cumulative, not linear in what you typed.

### 2. What counts as context

**Context is everything Claude sees on a given turn:**

- **The system prompt**
- **The message history**
- **Tool definitions and tool results**
- **Attached files and skills**
- **Thinking blocks**

> **It's the input to every single API call.**

#### Three consequences

|     |                                                                                       |
| --- | ------------------------------------------------------------------------------------- |
| 💸  | **You pay for it on the way in, and you pay for it on the way out**                   |
| 💥  | **Once the window is full, the request fails**                                        |
| 🎯  | **So the goal isn't to fit everything in — the goal is to fit the _right_ things in** |

### 3. The four patterns

**Anthropic publishes four patterns for managing context in long-running agents.**

> **Three are first-class API features, and one is a design pattern.**

| #   | Pattern                    | Type               | Fixes         |
| --- | -------------------------- | ------------------ | ------------- |
| 1   | **Just-in-time context**   | **Design pattern** | Window size   |
| 2   | **Server-side compaction** | API feature        | Window size   |
| 3   | **Prompt caching**         | API feature        | Cost          |
| 4   | **The memory tool**        | API feature        | Statelessness |

### 4. Pattern 1 — Just-in-time context

> **Don't load everything upfront. Load what the agent needs now, and let it pull more in via tools when it asks.**

> **This is the design pattern of the four: nothing special in the API, just a deliberate choice about what you load and when.**

### 5. Pattern 2 — Server-side compaction

> **When a conversation runs long, Anthropic's server-side compaction summarizes old turns into a single block.**

**You opt in by adding a `context_management` key to your request, holding an `edit` with a `type`:**

```python
response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=1024,
    context_management={
        "edits": [
            {"type": "compact"}
        ]
    },
    messages=messages,
)
```

#### The benefit

> **The API auto-summarizes when the input crosses the trigger threshold. You don't have to track conversation length yourself.**

```
  Before compaction              After compaction
  ─────────────────              ────────────────
  turn 1  ┐                      ┌──────────────────┐
  turn 2  │                      │ summary of turns │
  turn 3  ├── all verbatim  ──▶  │ 1–8 (one block)  │
  ...     │                      └──────────────────┘
  turn 8  ┘                      turn 9  ┐
  turn 9  ┐                      turn 10 ┘ verbatim
  turn 10 ┘
```

> **The trade-off to be aware of:** a summary is lossy. Old detail becomes gist. For most long conversations that's exactly what you want — but if some early fact must survive word-for-word, that belongs in memory (Pattern 4), not in the tail of a conversation.

### 6. Pattern 3 — Prompt caching

> **Prompt caching lets you mark the stable parts of a request — the system prompt, the tool definitions, a long document — and reuse them across calls at a fraction of the cost.**

**The key word is _stable_.** The parts that are byte-for-byte identical on every call are exactly the parts you shouldn't pay full price for repeatedly.

#### The math matters more than it looks

> **If your system prompt is 4,000 tokens and you call it 100 times an hour, caching is the difference between a usable bill and a phone call from finance.**

```
  4,000 tokens × 100 calls/hour = 400,000 tokens/hour
                                  …for text that never changed.
  × 24 hours = 9.6 million tokens a day, re-sent identically.
```

### 7. Pattern 4 — The memory tool

> **Some context needs to survive across sessions:** user preferences, the agent's running notes, what was decided last week.

**The recommended primitive for this is the memory tool.**

#### How it works

1. **Claude reads and writes to a memory directory via tool calls.**
2. **You implement the storage backend client-side** — **a file system, a database, an encrypted store, whatever you want.**
3. **Anthropic auto-injects a system instruction telling Claude to check the memory directory before starting work.**

> **Point 2 is the important one.** Memory isn't a black box Anthropic holds. **You own the storage**, so you decide where it lives, how it's encrypted, how long it's retained, and who can read it. That matters for anything with compliance or privacy requirements.
>
> **Point 3 is what makes it actually get used.** Without that injected instruction, Claude would have the tool but no reason to look. This makes checking memory the default behaviour rather than a lucky accident.

### 8. Layering the patterns

> **In a production app, you'll usually layer all four at once.**

**Example — the compliance review agent:**

> It **caches its system prompt and tool definitions** (Pattern 3), and **pulls building code sections in just in time via `lookup_building_code`** (Pattern 1).

#### How to choose

> **Each pattern handles a different failure mode: cost, window size, statelessness.**
>
> #### **Pick the ones that match what's breaking for you.**

| What's breaking                                    | Reach for                |
| -------------------------------------------------- | ------------------------ |
| **The bill is too high**                           | **Prompt caching**       |
| **Long conversations hit the window**              | **Compaction**           |
| **You're loading huge reference material upfront** | **Just-in-time context** |
| **The agent forgets across sessions**              | **The memory tool**      |

> They aren't alternatives — they're independent fixes for independent problems. Diagnose first, then apply.

---

---

# Module 4: Managed Agents

## What Are Managed Agents?

### 1. What it is

> **Claude Managed Agents is a suite of APIs for building and deploying agents at scale.**

**The workflow:**

1. **You define agents** with specific **tools, personas, and capabilities**
2. **You configure sandbox environments** with the right **packages and network controls**
3. **You fire off sessions** from your own application
4. **Claude does the work** inside an **isolated container** with **full file system access, bash execution, and web search**

> **Notice what you're _not_ doing:** running a server, managing containers, writing the loop, or keeping any of it alive. You describe the agent and press go.

### 2. The agent loop, hosted for you

> **Under the hood, this is an agent loop: Claude reasons, calls a tool, reads the result, and repeats until the job is done.**

**You already built this by hand in Lesson 4.**

> **If you've built agents before, you've probably written this kind of loop yourself. Managed agents takes that same loop and hosts it on Anthropic's infrastructure, so you don't have to run it.**

#### This completes the ladder the course has been building

**You'll find Managed Agents in its own section of the Claude Console.**

### 3. Example 1 — A Kanban board that does the work

Note: Kanban board means **"visualize work in progress."** **You drag a ticket from "to do" to "in progress," and the work starts automatically.**

**The setup:** a Kanban board sitting on top of managed agents. **You drag a ticket into "in progress," and that fires off a session automatically.**

**The ticket reads: _"optimize website performance."_**

#### What happens

1. **Your back end creates a session.**
2. **The session points to an environment you configured** with **Lighthouse and Puppeteer pre-installed.**
3. **Your GitHub repo gets mounted into the container.**

> **That's the environment idea in action:** the sandbox isn't generic. You decided ahead of time that this kind of work needs Lighthouse and Puppeteer, so they're already there when the session starts.

**Now Claude has the codebase, the tools, and a rubric that defines what _done_ looks like:**

- **Lighthouse score above 90**
- **No render-blocking resources**
- **All images lazy loaded**

#### The work

**Claude runs the audit, then starts compressing images, inlining CSS, and deferring scripts.**

> **Every tool call streams back to the board in real time through the event stream, so you can watch the work as it happens.**

#### Then the rubric kicks in

> **A separate grader, running in its own context window, evaluates the output against your criteria. Claude reads that feedback, goes back in, fixes what it missed, and resubmits.**
>
> **In the demo, that loop takes the Lighthouse score up to 96.**

```
   Claude works  ──▶  GRADER (separate context)  ──▶  feedback
        ▲                                                │
        └────────────  fixes what it missed  ◀───────────┘

   repeat until the rubric passes → 96
```

> **Why the grader runs in its own context window matters:** It doesn't know or care about the reasoning behind the answer. It only looks at the final result and checks how good it is.
> This makes the evaluation more fair and objective than simply asking Claude, “Do you think your own work is good?”

#### Parallel sessions

> **You can drag a second ticket over while the first is still running. Two sessions, two containers, two separate tasks running in parallel.**

Separate containers means they can't interfere with each other.

### 4. Example 2 — A recurring research agent with memory

**A different shape of agent:** one that **tracks prices and plan changes across every SaaS tool your company pays for, with a report ready before stand-up.**

#### On each run, the agent

| Step                                                                                                                              | What it uses            |
| --------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| **Searches the web** for current pricing pages, checks for plan tier changes, flags new features that might affect your contracts | **Web search** (L7)     |
| **Runs a cost analysis in Python** inside the sandbox                                                                             | **Code execution** (L7) |
| **Uses an Excel spreadsheet skill** and writes an executive summary                                                               | **Skills** (L8)         |
| **Posts a link to Slack and creates a review task in Jira**                                                                       | **MCP servers** (L9)    |

> **Look at what just happened:** almost every feature from the previous lessons appears in one agent. Managed agents is where they all compose.

#### The memory part is the interesting bit

> **The agent also reads from and writes to a memory store. Before it starts, it checks what it found last week. After it finishes, it stores what changed.**

**The payoff:**

> **So next Monday's report can say _"compute costs are 15% lower since last week"_ instead of listing the same static pricing data every time.**

```
  WITHOUT memory                  WITH memory
  ──────────────                  ───────────
  Week 1: here is the pricing     Week 1: here is the pricing
  Week 2: here is the pricing     Week 2: compute is 15% cheaper
  Week 3: here is the pricing             than last week ← insight
          ↑ same report forever
```

> **A report that can't remember can only describe. A report that remembers can compare — and comparison is where the actual value is.**

### 5. Example 3 — Incident response with multiple agents

**The trigger:** **an alert fires from your monitoring stack.** **A custom tool on your back end receives the alert payload and sends it into a new session as a tool result.**

### Multi-agent coordination

1. **A coordinator agent receives the alert and delegates to three specialists.**
2. **Each specialist runs in its own context window on the same shared file system.**
3. **The specialists report back, and the coordinator synthesizes their findings into a single incident summary.**

```
                  ALERT
                    │
            ┌───────▼────────┐
            │  COORDINATOR   │
            └───┬────┬───┬───┘
       delegates │    │   │
          ┌──────┘    │   └──────┐
          ▼           ▼          ▼
      specialist  specialist  specialist
      (own ctx)   (own ctx)   (own ctx)
          └──────┐    │   ┌──────┘
                 ▼    ▼   ▼
            reports back to coordinator
                    │
            single incident summary
```

> **"Own context window, shared file system"** is the key design: each specialist gets clean headroom to dig into its own angle (no context pollution from the others), but they all see the same files, so they're investigating the same incident, not three copies of it.

#### The permissions policy

> **Before the summary goes to Slack, the permissions policy fires. You see the draft on screen, approve it, and the message goes out.**
>
> #### **Sensitive actions wait for a human.**

> **This is the guardrail worth noticing.** An autonomous agent that can post to Slack during an incident is a liability without this. The agent does all the work; a person approves the part that leaves the building.

#### Memory ties it together

> **The coordinator checks past incidents in the memory store and flags a pattern:** _"this looks like the DNS resolution issue from two weeks ago that was caused by a misconfigured TTL."_
>
> **The next time a similar alert fires, the agent starts with that context instead of diagnosing from scratch.**

An agent with memory **gets better at your specific system over time** — not because the model changed, but because its notes accumulated.

---

## Building Your First Managed Agent — Notes

### 1. When to stop running the loop yourself

**If you've built an agent loop by hand, you know the drill:** `while` loops, stop reason switches, tool executions.

> **That works, and for a lot of features it's actually the right shape.**

**But sometimes that loop is going to run for a very long time** — **minutes, maybe even hours** — across **many tools**, with:

- **state to keep**
- **files to write**
- **work to resume after a network hiccup**

> **At that point, you don't want to run the loop on your server. You want to delegate it.**

> **The word to notice is _resume_.** A 90-minute job on your own server is a 90-minute window in which a deploy, a restart, or a dropped connection destroys everything done so far. That's the real reason to hand the loop over, more than convenience.

### 2. What a managed agent is

> **A managed agent is an agent loop that runs on Anthropic's infrastructure instead of yours.**

**The flow:**

1. **You describe the agent once**
2. **You give it an environment to work in**
3. **You start a session**
4. **Anthropic runs the loop**
5. **You just stream the events back out as it works**

> ✅ **Managed agents are enabled by default for every API account — no special access needed.**
>
> (Unlike Skills and MCP in earlier lessons, there's no beta header to add here.)

### 3. The four primitives

> **There are four primitives, and they come in order:**

| #   | Primitive       | What it is                                                                                         |
| --- | --------------- | -------------------------------------------------------------------------------------------------- |
| 1   | **Agent**       | **The persona: model, system prompt, and toolset.** **Reusable across many runs**                  |
| 2   | **Environment** | **Where the agent runs:** cloud or local, networking config, and so on                             |
| 3   | **Session**     | **A single run of an agent inside a certain environment.** **The session is the unit of work**     |
| 4   | **Events**      | **The messages flowing in and out:** the agent's actions, the tool calls, the results, the replies |

#### How the pieces fit together

> **Your app talks to a session, the session drives work inside the environment, and everything that happens flows back out through the event stream.**

```
   your app  ──────▶  SESSION  ──────▶  ENVIRONMENT
                         │              (the sandbox where
                         │               work happens)
                         ▼
                   EVENT STREAM  ──────▶  back to your app
```

#### ⚠️ The mental shift

> ### **You're not running a `while` loop. You're sending events and reading events.**

```
  MANUAL LOOP (L4)                MANAGED AGENT
  ───────────────                 ─────────────
  while True:                     send an event in
      call the API                read events out
      check stop_reason           (the loop is elsewhere)
      run the tool
      append results
```

**Reusable vs single-use:** the **agent** and the **environment** are definitions you create once. The **session** is one run. Same agent + same environment → many sessions.

### 4. The smallest possible managed agent

**The task:** **create a file in the temp drive, count its lines, and report back.**

**For tools**, we use the **agent toolset** — **Anthropic's bundled file, bash, and web tools.**

> **They work fine for this task, so we don't have to define any tools ourselves.**

#### Step 1 — Create the agent

```python
import anthropic

client = anthropic.Anthropic()

agent = client.beta.agents.create(
    name="Line Counter",
    model="claude-opus-4-8",
    system="You are a helpful agent that completes small file tasks.",
    tools=[
        {"type": "agent_toolset_20260401", "default_config": {"enabled": True}}
    ],
)
```

**Note the agent toolset defined right in the `tools` array — that's the bundled toolset.**

> **Remember: the agent is reusable. Create it once and run it across many sessions.**

#### Step 2 — Create the environment

```python
environment = client.beta.environments.create(
    name="line-counter-env",
    config={
        "type": "cloud",
        "networking": {"type": "unrestricted"},
    },
)
```

> **This spins up the container template — cloud, with unrestricted networking. This is the sandbox where the file actually gets written.**

#### Step 3 — Create the session

```python
session = client.beta.sessions.create(
    agent=agent.id,
    environment_id=environment.id,
    title="Count lines demo",
)
```

**The session joins an agent to an environment,** plus **an optional title**.

#### Step 4 — Open the stream, THEN send the kickoff

> ### ⚠️ **Order matters here.**

```python
with client.beta.sessions.events.stream(session_id=session.id) as stream:
    # Stream is open — now send the kickoff
    client.beta.sessions.events.send(
        session_id=session.id,
        events=[
            {
                "type": "user.message",
                "content": [
                    {
                        "type": "text",
                        "text": "Create a file in the temp directory, "
                                "count its lines, and report back.",
                    }
                ],
            }
        ],
    )
```

> **The stream only delivers events that occur after it opens, so always open it before sending the kickoff message.**

**Also notice: it's `events` — plural.**

> **Events are how everything flows in this API.**

#### Step 5 — Consume the stream

**Three event types matter for this demo:**

| Event                     | Meaning                     |
| ------------------------- | --------------------------- |
| **`agent.message`**       | **Claude's text**           |
| **`agent.tool_use`**      | **What tool Claude picked** |
| **`session.status_idle`** | **The agent is done**       |

```python
for event in stream:
    if event.type == "agent.message":
        for block in event.content:
            if block.type == "text":
                print(block.text, end="", flush=True)
    elif event.type == "agent.tool_use":
        print(f"\n[tool] {event.name}")
    elif event.type == "session.status_idle":
        print("\n--- Agent done ---")
        break
```

> **`session.status_idle` is the managed-agent equivalent of `stop_reason == "end_turn"`** from Lesson 4 — it's your signal to stop listening.
>
> **And `agent.message` content is still an array of blocks** you loop and type-check. That rule from Lesson 2 has now held for every single feature in the course.

#### What you see when you run it

> **The output is the agent reasoning out loud — actual text, the tools it picks, and a final answer.**
>
> **All of it running inside Anthropic's container, not yours.**

---

### 5. The trade

> **Usually with agents, we have our own loop where we have to control everything.**
>
> **With managed agents, you delegate that loop, the sandbox, and the resumability — and just consume the event stream as it comes in.**

---

---

# Module 5: Building with Claude Code

## Building with Claude Code

### 1. The idea

**Writing code that calls the Claude API by hand works fine** — you've done it for twelve lessons.

> **But there's an even faster path: have Claude write it for you.**

**In this lesson:** use **Claude Code** to **fill in an API integration from a stubbed-out file** — using **the same primitives you've learned throughout this course.**

### 2. Starting from a stub

**The project is simple: a TypeScript file that gets weather.** It contains **two stubs**:

| Stub             | Should do                                                     |
| ---------------- | ------------------------------------------------------------- |
| **`getWeather`** | **Accepts a city and returns the temperature and conditions** |
| **`run`**        | **Uses the tool runner and the Claude TypeScript SDK**        |

> **Reminder from Lesson 5:** **the tool runner is the piece that handles tool calling and the agent loop for you, so you don't have to wire that up manually.**

> **Why start from a stub rather than an empty file?** The stub carries the **function signatures and types**. That's a precise specification of what you want — far more exact than describing it in a paragraph of English. Claude Code then has something concrete to write _against_.

### 3. The Claude API skill

> **Claude Code comes with a built-in skill called Claude API.**

**Two ways it activates:**

1. **You invoke it directly with `/claude-api`**
2. **Claude Code invokes it automatically when it detects that you're using the TypeScript SDK**

#### If you don't see the skill

```
/plugin marketplace add AnthropicsSkills
```

> ⚠️ **Note the `s` at the end of `Anthropics` — it's easy to miss.**

### 4. One prompt, working code

**Open the project folder in your terminal and launch Claude Code.**

> **From there, it takes a single prompt.**

#### What makes a good prompt — three things

| The prompt should name           | Why                                                                      |
| -------------------------------- | ------------------------------------------------------------------------ |
| **1. The file you want changed** | No ambiguity about where to work                                         |
| **2. The pattern you want used** | e.g. "use the tool runner" — otherwise you might get a hand-written loop |
| **3. The end state you expect**  | What "done" looks like, so it can check itself                           |

#### What Claude Code then does

1. **Fills in `getWeather` and `run` against the types**
2. **Appends a call at the bottom of the file**
3. **Executes the script**
4. **Reports the output**
5. **If something errors out, it reads the error message and patches the code in place**

> **Step 5 is the part that makes it an agent rather than a code generator.** It doesn't hand you code and wish you luck — it runs the thing and fixes what breaks. That's the agent loop.

### 5. The pattern to remember

> **Most of what you write against the Claude API has a familiar shape:**

```
   1. Define a tool
   2. Hand it to a runner
   3. Return the result
```

### And the working habit that follows from it

> **You don't need to type that from memory every single time.**
>
> ### **Instead: stub the file, hand it to Claude Code, and just review the diff.**

| Step         | You do                         | Claude Code does                   |
| ------------ | ------------------------------ | ---------------------------------- |
| **Stub**     | Write the signatures and types | —                                  |
| **Delegate** | Write one good prompt          | Fills it in, runs it, fixes errors |
| **Review**   | **Read the diff**              | —                                  |
