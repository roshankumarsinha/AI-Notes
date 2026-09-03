# Tokenization & Byte Pair Encoding (BPE)

## What Tokenization Is

A model doesn't understand text — it understands numbers. So before any text reaches the neural network, it has to be chopped into pieces, and each piece mapped to an integer ID. Those pieces are called **tokens**, and the chopping step is **tokenization**.

The pipeline looks roughly like this:

```
"I love pizza"  →  ["I", " love", " pizza"]  →  [40, 1842, 14268]  →  model
     text              tokens                        token IDs
```

The model has a fixed dictionary called the **vocabulary** — a lookup table mapping every possible token to an ID (and back). Each ID then points to an **embedding**, a vector the network actually does math on. Tokenization is just the front door.

The whole design question is: _what should a token be?_ That's where it gets interesting.

## The Three Naive Approaches (and Why They Fail)

### 1. Word-level

One token per word. `"I love pizza"` → `["I", "love", "pizza"]`. Feels natural, but breaks badly:

- **Vocabulary explodes.** English has millions of word forms (run, runs, running, ran, runner…). You can't fit them all.
- **Wastes structure.** The model can't see that "running" and "runs" share "run"; they're just two unrelated IDs.

### 2. Character-level

One token per character. `"pizza"` → `["p","i","z","z","a"]`. This fixes the vocabulary problem — you only need ~100 symbols and _nothing_ is ever unknown. But:

- Sequences become **extremely long** (a 1000-word doc is thousands of tokens), which is slow.
- Each token carries almost **no meaning** on its own, so the model works much harder to learn anything.

### 3. Subword-level(Mostly used)

The compromise everyone actually uses. Keep common words or actual meaning words whole ("the", "pizza"), but break rare words into meaningful chunks:

```
"tokenization"  →  ["token", "ization"]
"unhappiness"   →  ["un", "happiness"]
```

Sometimes, english words are broken into pieces to extract meaning. Like "polymorphism" → ["poly", "morph", "ism"]. The model sees the shared pieces and can generalize better.

Common things stay compact, rare things stay representable, and shared pieces ("token", "un") get reused across many words. **Byte Pair Encoding is the most famous way to learn these subword pieces automatically from data.**

## Byte Pair Encoding (BPE) — The Idea

> Start with individual characters. Then repeatedly find the most frequent adjacent pair and merge it into a new symbol. Keep doing this until you've built a vocabulary of the size you want.

Frequent pairs get merged first, so common combinations ("t"+"h" → "th", "th"+"e" → "the") become single tokens, while rare words stay broken into smaller pieces. The algorithm _discovers_ useful subwords from data instead of you hand-listing them.

There are two phases:

1. **Training** — learn a set of _merge rules_ from a corpus (done once).
2. **Encoding** — apply those merge rules, in order, to tokenize any new text.

## The BPE Algorithm — Worked Example

Let's train on a tiny corpus. After counting our text, we have these words with their frequencies:

| Word | Count |
| ---- | ----- |
| hug  | 10    |
| pug  | 5     |
| pun  | 12    |
| bun  | 4     |
| hugs | 5     |

### Step 0 — Start from characters

Split every word into characters. Starting vocabulary is just the unique characters: `b, g, h, n, p, s, u`.

```
h u g       (×10)
p u g       (×5)
p u n       (×12)
b u n       (×4)
h u g s     (×5)
```

### Step 1 — Count every adjacent pair (weighted by word frequency)

| Pair | Where it appears           | Count  |
| ---- | -------------------------- | ------ |
| u,g  | hug(10) + pug(5) + hugs(5) | **20** |
| p,u  | pug(5) + pun(12)           | 17     |
| u,n  | pun(12) + bun(4)           | 16     |
| h,u  | hug(10) + hugs(5)          | 15     |
| g,s  | hugs(5)                    | 5      |
| b,u  | bun(4)                     | 4      |

Winner: **(u, g)** with 20. Create merge rule `u + g → ug` and apply everywhere:

```
h ug        (×10)
p ug        (×5)
p u n       (×12)
b u n       (×4)
h ug s      (×5)
```

### Step 2 — Recount pairs

| Pair | Count                     |
| ---- | ------------------------- |
| u,n  | pun(12) + bun(4) = **16** |
| h,ug | hug(10) + hugs(5) = 15    |
| p,u  | 12                        |
| p,ug | 5                         |
| ug,s | 5                         |
| b,u  | 4                         |

Winner: **(u, n)** = 16. New rule `u + n → un`:

```
h ug        (×10)
p ug        (×5)
p un        (×12)
b un        (×4)
h ug s      (×5)
```

### Step 3 — Recount again

| Pair | Count           |
| ---- | --------------- |
| h,ug | 10 + 5 = **15** |
| p,un | 12              |
| p,ug | 5               |
| ug,s | 5               |
| b,un | 4               |

Winner: **(h, ug)** = 15. New rule `h + ug → hug`:

```
hug         (×10)
p ug        (×5)
p un        (×12)
b un        (×4)
hug s       (×5)
```

You keep going until you hit your target vocabulary size (in real systems, tens of thousands of merges). After these three steps, our learned merge rules **in order** are:

```
1. u + g  → ug
2. u + n  → un
3. h + ug → hug
```

That ordered list _is_ the trained tokenizer. Notice how the algorithm organically discovered that "hug", "ug", and "un" are useful reusable units — nobody told it that.

## Encoding New Text With the Learned Rules

To tokenize a new word, split it into characters and apply the merge rules **in the same order they were learned**.

**"hugs"** → `h u g s`

- Rule 1 (u+g→ug): `h ug s`
- Rule 2 (u+n): doesn't apply
- Rule 3 (h+ug→hug): `hug s`
- **Result: `["hug", "s"]`** — two tokens ✅

**"bug"** → `b u g`

- Rule 1: `b ug`
- Rules 2, 3: don't apply
- **Result: `["b", "ug"]`** — never seen whole, still represented sensibly from known pieces.

This is the payoff: a word the tokenizer never saw during training still gets encoded, not thrown away.

## An Important Wrinkle: Totally Unseen Characters

Try **"mug"**. There's no "m" in our vocabulary (it never appeared in training), so classic BPE would produce an `[UNK]`. That's a real weakness.

## Code Walkthrough (JavaScript)

Here is a minimal, complete BPE implementation, followed by a line-by-line explanation with the values traced at every step.

```javascript
// Simple Byte Pair Encoding
let corpus = { hug: 10, pug: 5, pun: 12, bun: 4, hugs: 5 };

// Split each word into characters
let words = {};
for (let word in corpus) words[word] = word.split("");

let merges = [];

// Train: do 3 merges
for (let step = 0; step < 3; step++) {
  // Count all adjacent pairs
  let counts = {};
  for (let word in words) {
    let s = words[word];
    for (let i = 0; i < s.length - 1; i++) {
      let pair = s[i] + "," + s[i + 1];
      counts[pair] = (counts[pair] || 0) + corpus[word];
    }
  }

  // Find the most frequent pair
  let best = null,
    max = 0;
  for (let pair in counts) {
    if (counts[pair] > max) {
      max = counts[pair];
      best = pair;
    }
  }

  let [a, b] = best.split(",");
  merges.push([a, b]);
  console.log(`merge ${a} + ${b} -> ${a + b}`);

  // Apply the merge to every word
  for (let word in words) {
    let s = words[word],
      out = [];
    for (let i = 0; i < s.length; i++) {
      if (s[i] === a && s[i + 1] === b) {
        out.push(a + b);
        i++;
      } else out.push(s[i]);
    }
    words[word] = out;
  }
}

// Encode a new word using the learned merges
function encode(word) {
  let s = word.split("");
  for (let [a, b] of merges) {
    let out = [];
    for (let i = 0; i < s.length; i++) {
      if (s[i] === a && s[i + 1] === b) {
        out.push(a + b);
        i++;
      } else out.push(s[i]);
    }
    s = out;
  }
  return s;
}

console.log(encode("hugs")); // [ 'hug', 's' ]
console.log(encode("bug")); // [ 'b', 'ug' ]
```

### Setup

```javascript
let corpus = { hug: 10, pug: 5, pun: 12, bun: 4, hugs: 5 };
```

Our training data: five words and how often each appears. The counts matter because BPE merges the pair that's most frequent _across the whole corpus_.

```javascript
let words = {};
for (let word in corpus) words[word] = word.split("");
```

`word.split("")` breaks a string into individual characters, so this turns every word into an array of single letters:

```javascript
{
  hug:  ["h", "u", "g"],
  pug:  ["p", "u", "g"],
  pun:  ["p", "u", "n"],
  bun:  ["b", "u", "n"],
  hugs: ["h", "u", "g", "s"]
}
```

The keys stay the original words (we still need them to look up frequency in `corpus`); the values are character arrays we'll gradually merge.

```javascript
let merges = [];
```

Collects the merge rules we learn, **in order** — order is the whole point.

### The training loop

```javascript
for (let step = 0; step < 3; step++) {
```

We do 3 merges total. Each pass learns exactly one merge rule. (Real tokenizers loop thousands of times; 3 is enough to demonstrate.)

**Part 1 — count all adjacent pairs**

```javascript
let counts = {};
for (let word in words) {
  let s = words[word];
  for (let i = 0; i < s.length - 1; i++) {
    let pair = s[i] + "," + s[i + 1];
    counts[pair] = (counts[pair] || 0) + corpus[word];
  }
}
```

For each word we look at every neighboring pair. `s[i] + "," + s[i+1]` builds a key like `"u,g"` (the comma is just a separator). The key line is `+ corpus[word]` — we don't count each pair as 1, we add the word's **frequency**. A pair inside "pun" counts 12 times because "pun" appears 12 times. `(counts[pair] || 0)` means "if unseen, start from 0."

Tracing the first pass:

| Word | Freq | Pairs it contains | Contribution             |
| ---- | ---- | ----------------- | ------------------------ |
| hug  | 10   | u,g and h,u       | u,g +10 · h,u +10        |
| pug  | 5    | p,u · u,g         | p,u +5 · u,g +5          |
| pun  | 12   | p,u · u,n         | p,u +12 · u,n +12        |
| bun  | 4    | b,u · u,n         | b,u +4 · u,n +4          |
| hugs | 5    | h,u · u,g · g,s   | h,u +5 · u,g +5 · g,s +5 |

Totals:

```javascript
{ "u,g": 20, "h,u": 15, "p,u": 17, "u,n": 16, "b,u": 4, "g,s": 5 }
```

**Part 2 — find the most frequent pair**

```javascript
let best = null,
  max = 0;
for (let pair in counts) {
  if (counts[pair] > max) {
    max = counts[pair];
    best = pair;
  }
}
let [a, b] = best.split(",");
merges.push([a, b]);
console.log(`merge ${a} + ${b} -> ${a + b}`);
```

A standard "find the maximum" loop. `"u,g"` at 20 wins. `best.split(",")` splits it back into `a = "u"`, `b = "g"`. We record `["u","g"]` and print `merge u + g -> ug`.

**Part 3 — apply the merge to every word**

```javascript
for (let word in words) {
  let s = words[word],
    out = [];
  for (let i = 0; i < s.length; i++) {
    if (s[i] === a && s[i + 1] === b) {
      out.push(a + b);
      i++;
    } else out.push(s[i]);
  }
  words[word] = out;
}
```

We rewrite every word, fusing the winning pair wherever it appears. Scanning left to right into a fresh `out` array: if the current token is `a` **and** the next is `b`, push the combined `a+b` and do `i++` (that extra increment skips the token we just consumed); otherwise push the current token unchanged. After merging `u+g`:

```javascript
{
  hug:  ["h", "ug"],
  pug:  ["p", "ug"],
  pun:  ["p", "u", "n"],
  bun:  ["b", "u", "n"],
  hugs: ["h", "ug", "s"]
}
```

**The next two passes** repeat on the updated state:

- **Step 2** — winner `u,n` at 16 (pun 12 + bun 4). Rule `u+n → un`.
- **Step 3** — winner `h,ug` at 15 (hug 10 + hugs 5). Rule `h+ug → hug`.

After the loop, `merges` holds the three rules **in learned order** — this ordered list _is_ the trained tokenizer:

```javascript
[
  ["u", "g"],
  ["u", "n"],
  ["h", "ug"],
];
```

### The encode function

```javascript
function encode(word) {
  let s = word.split("");
  for (let [a, b] of merges) {
    let out = [];
    for (let i = 0; i < s.length; i++) {
      if (s[i] === a && s[i + 1] === b) {
        out.push(a + b);
        i++;
      } else out.push(s[i]);
    }
    s = out;
  }
  return s;
}
```

To tokenize a new word, split into characters, then replay every learned merge **in the same order**. The inner loop is the same fuse-the-pair logic from training.

Order matters: rule 3 (`h + ug → hug`) can only fire if `ug` already exists as one token — which only happens because rule 1 (`u + g → ug`) ran first. Replaying out of order would break the chain.

Tracing `encode("hugs")`:

| Stage         | State of `s`                             |
| ------------- | ---------------------------------------- |
| Start (split) | `["h", "u", "g", "s"]`                   |
| Rule 1 `u+g`  | `["h", "ug", "s"]`                       |
| Rule 2 `u+n`  | `["h", "ug", "s"]` — no `u,n`, unchanged |
| Rule 3 `h+ug` | `["hug", "s"]`                           |

Result: `["hug", "s"]`

Tracing `encode("bug")`:

| Stage         | State of `s`                                  |
| ------------- | --------------------------------------------- |
| Start         | `["b", "u", "g"]`                             |
| Rule 1 `u+g`  | `["b", "ug"]`                                 |
| Rule 2 `u+n`  | `["b", "ug"]` — unchanged                     |
| Rule 3 `h+ug` | `["b", "ug"]` — no `h` before `ug`, unchanged |

Result: `["b", "ug"]`

"bug" was never in the training corpus, yet it still tokenizes cleanly into known pieces (`b` and `ug`). That's the entire value of BPE — unseen words don't break, they decompose into learned subwords.

### One subtle detail

In the condition `s[i] === a && s[i + 1] === b`, when `i` is the last index, `s[i+1]` is `undefined`. That's harmless: `undefined === b` is just `false`, so it correctly doesn't merge. Nothing to fix — JavaScript's handling of out-of-bounds access happens to do the right thing here.

---

# Vectorization (Word Embeddings) & HNSW

Vectorization is what happens _after_ tokenization: the integer token IDs get turned into vectors whose geometry carries meaning. These notes focus on the approach that made meaning into math — **word embeddings** — and then on **HNSW**, the algorithm used to search millions of those vectors fast.

## What Vectorization Is

Tokenization gives us integer IDs. But an ID like `14268` for "pizza" is meaningless as a number — it's just a slot in a lookup table. `14268` isn't "bigger" or "closer to" `14269` in any real sense. What we actually want is to represent each piece of text as a **vector**: a list of numbers where the _geometry_ carries meaning. Similar things should land near each other in space; different things should land far apart.

> **Vectorization** is the process of turning text into numeric vectors so that mathematical operations — especially measuring distance and similarity — reflect real-world meaning.

Once text is vectors, "find me documents similar to this one" becomes "find the vectors nearest to this vector." That single idea powers semantic search, recommendations, and RAG systems.

The whole design question is: _how do we choose the numbers so that distance means similarity?_ Word embeddings answer it beautifully.

## The Core Problem

AI can't read letters and derive meaning the way humans do. The only thing a machine can actually work with is **numbers** — so every word has to be converted into a list of numbers, called a **vector**. This isn't an optimization; it's a hard requirement.

## Words Become Positions in a "Meaning Space"

Each word gets mapped to a point in a multi-dimensional space, arranged by meaning:

- **Similar words end up close together.**
- **Different words end up far apart.**
  The clever part: this turns a _meaning_ question ("are these two words related?") into a _geometry_ question ("are these two points near each other?") — and geometry is something a computer can answer with arithmetic.

## What the Numbers Inside a Vector Mean

Each value in the vector represents a **hidden feature** of the word — with two important qualifiers:

1. **They're learned, not designed.** Nobody decided "dimension 47 = sweetness." These features emerge automatically from exposure to massive amounts of text.
2. **They're not all human-readable.** Some dimensions match something you could name (like "edibility"), others are abstract and correspond to nothing a human has a word for.
   **Worked example — apple, banana, car:**

- **Apple and banana** have similar values on dimensions like _edibility_ and _sweetness_ → they're close.
- **Car** is completely different — it's in an entirely different category → far away.
  _(Note: "edibility" and "sweetness" are just illustrations of what a learned dimension_ could _look like — a real model doesn't literally have a labelled "sweetness" axis.)_

The payoff: because these features were _learned from data_ rather than hand-written as rules, the system can discover relationships **nobody explicitly programmed.** It isn't limited to connections someone remembered to write down — unlike a dictionary or a hand-built rulebook.

## Measuring Similarity: Cosine Similarity

How do you measure how similar two word-vectors are? **Cosine similarity** — and the key detail:

> It measures the **angle** between two vectors, not the distance between them. The **smaller the angle, the more similar** the words.

Think of each vector as an arrow pointing out from a center. Two arrows pointing _the same direction_ are similar, regardless of their length.

**The example, with scores:**

| Word pair      | Cosine similarity | Meaning            |
| -------------- | ----------------- | ------------------ |
| Apple ↔ Banana | **0.98**          | Strongly related   |
| Apple ↔ Car    | **~0.1**          | Almost no relation |

The takeaway is the _spread_, not the exact digits (which depend on the model): near **1.0** = strongly related, near **0** = effectively unrelated.

## Where It's Used

- **Finding related words** — retrieve the words nearest a given one.
- **Search by meaning** — match a query to results by _meaning_, not just matching exact words.
- **Recommendations** — surface items similar to ones you liked.
  Named examples: **ChatGPT**, **Google Search**, and **Netflix recommendations**. (Netflix quietly extends the idea beyond words — the same vector-plus-angle trick works on _anything_ you can turn into a vector, like films and viewing habits.)

## Word Embeddings — The Leap to _Meaning_

> **The distributional hypothesis:** a word is defined by the company it keeps. Words that appear in similar contexts should get similar vectors.

"cat" and "dog" both show up near "pet", "feed", "vet", "cute" — so a model that learns from context naturally pushes their vectors close together. The famous payoff is that meaning becomes **arithmetic**:

```
vector("king") − vector("man") + vector("woman") ≈ vector("queen")
```

## Word2Vec: Two Flavors

Word2Vec learns embeddings by turning "understand language" into a simple prediction game played over a sliding window of text. There are two directions to play it:

- **CBOW (Continuous Bag of Words):** given the _surrounding context words_, predict the _center word_. Faster, good for frequent words.
- **Skip-gram:** given the _center word_, predict the _surrounding context words_. Slower, but better for rare words and generally higher quality.

We'll walk through **Skip-gram**, since it's the more widely used and the mechanics are clearest.

The trick that makes it efficient in practice is **negative sampling**: instead of updating the entire vocabulary every step, we nudge the vectors for _one real (word, context) pair_ to be closer, and a _handful of random "negative" pairs_ to be farther apart. Each word actually gets **two** vectors during training — one for when it's the _center_ word (input) and one for when it's a _context_ word (output). After training we usually keep the input vectors.

## Measuring Similarity Between Vectors

Vectors are useless until you can compare them. The standard tool for embeddings is **cosine similarity** — the cosine of the angle between two vectors. It ignores length and asks only "do these point the same direction?"

```
cosine(A, B) = (A · B) / (|A| × |B|)
```

`A · B` is the dot product (multiply matching components, sum them); `|A|` is the vector's length. The result runs from **1** (identical direction) through **0** (unrelated) to **−1** (opposite).

Comparing **king** and **queen** from below:
Illustrative 2-D vectors, but the same math works in 300-D or N-D.

```
king  = [0.9, 0.7]
queen = [0.8, 0.9]
man   = [0.7, 0.2]
woman = [0.6, 0.4]
```

```
A · B  = (0.9×0.8) + (0.7×0.9) = 0.72 + 0.63 = 1.35
|king| = √(0.9² + 0.7²) = √1.30 = 1.140
|queen|= √(0.8² + 0.9²) = √1.45 = 1.204

cosine = 1.35 / (1.140 × 1.204) = 1.35 / 1.373 = 0.98
```

A cosine of **0.98** — nearly identical direction — confirms "king" and "queen" are highly related. The geometry now matches intuition, which is the entire goal of vectorization.

## The Scaling Problem → Why HNSW Exists

Now the real-world catch. Suppose you've vectorized 100 million documents. A query arrives as a vector, and you want its 10 nearest neighbors. The obvious method — **brute force**: compute the distance to all 100 million and sort — is `O(N)` per query. Far too slow at scale.

We need **Approximate Nearest Neighbor (ANN)** search: give up _guaranteed_ perfect results in exchange for enormous speed, finding the _almost_-nearest neighbors in roughly `O(log N)` time. **HNSW (Hierarchical Navigable Small World)** is the most popular ANN algorithm — it powers vector databases like Pinecone, Qdrant, Weaviate, Milvus, and FAISS.

## HNSW — The Two Ideas It Combines

HNSW builds a **graph** where each vector is a node connected to nearby vectors. To search, you hop along edges toward the query. Two ingredients make this fast.

**Idea 1 — "Small World" graphs (navigable by greedy hops).** In a small-world network, any node reaches any other in very few hops ("six degrees of separation"). Each point connects to a mix of close neighbors _and_ a few longer-range links. To find something, you **greedily** walk to whichever neighbor is closer to your target, repeating until no neighbor is closer. Like navigating a city: highways to the right district, then local streets to the exact address.

**Idea 2 — Hierarchy (layers, like a skip list).** A single graph still wastes hops, so HNSW stacks **multiple layers**:

- The **top layer** is sparse — few nodes, long-range links. Covers huge distances in one jump.
- Each layer **down** gets denser, with more nodes and shorter links.
- The **bottom layer (layer 0)** holds _every_ node with fine-grained local links.

## The HNSW Search Algorithm, Walked Through

**Goal:** find the nearest neighbor to a query vector **Q**.

1. **Enter at the top layer** at a fixed entry point.
2. **Greedy search on this layer:** look at the current node's neighbors; if any is closer to Q, move there. Repeat until no neighbor is closer — the local best on this layer.
3. **Drop down one layer**, using that best node as the new start.
4. **Repeat** the greedy search on the denser layer — finer moves now possible.
5. At **layer 0**, do a final greedy search but keep the _best `ef` candidates_ seen (not just the single closest), where `ef` is a tunable knob. Return the top `k`.

In one line: HNSW gives roughly **logarithmic search time** and excellent recall, at the cost of **higher memory** (storing all those links) and a slower one-time build. For most vector-search workloads that's a great deal — which is why it's the default in nearly every vector database.

## Quick Summary

| Concept           | One-liner                                                      |
| ----------------- | -------------------------------------------------------------- |
| Vectorization     | Turning text into vectors where distance = similarity          |
| Cosine similarity | Angle between vectors; 1 = same direction, 0 = unrelated       |
| ANN               | Approximate nearest neighbor — trade exactness for speed       |
| HNSW              | Layered small-world graph; greedy top-down search in ~O(log N) |
