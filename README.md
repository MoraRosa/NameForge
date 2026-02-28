# NameForge

> AI-powered startup name generator with live domain checking and a built-in transformer explainer. One file. No frameworks. No API keys.

**[Live Demo →](https://morarosa.github.io/NameForge)**

---

## What it is

NameForge is a browser-based name generator that runs a real character-level transformer — the same fundamental architecture as GPT-4, built from scratch in ~200 lines of vanilla JavaScript. Describe your product, pick a creative vibe, and the model trains on a corpus of 200+ real SaaS names and generates candidates tailored to your input.

Every generated name is instantly checked for domain availability across 8 TLDs, live, in-browser, with no proxy and no API key — using the official RDAP protocol queried directly against authoritative registry servers.

A full educational section explains how the underlying technology works, from tokenisation through to backpropagation and sampling — accessible to curious non-technical readers, precise enough for developers learning to build.

---

## Features

### Name Generation
- Character-level transformer trained live in the browser on each run
- 200+ real SaaS and startup names as the training corpus (Stripe, Notion, Linear, Vercel, Supabase, etc.)
- 4 vibe modes — **Bold & Powerful**, **Clean & Modern**, **Playful & Weird**, **Abstract & Rare** — each with distinct temperature settings, seed prefixes, and suffix biases
- Keyword extraction from your product description seeds the generation
- Name scoring by length, phonetics, character variety, and vibe fit
- Up to 8 deduplicated, ranked candidates per run

### Domain Availability
- Live RDAP lookups across `.com`, `.io`, `.co`, `.app`, `.dev`, `.ai`, `.net`, `.org`
- Results stream in parallel as each registry responds — no waiting for the slowest
- Hits authoritative registry servers directly (Verisign for `.com`/`.net`, Google Registry for `.app`/`.dev`, etc.)
- Automatic fallback via `rdap.org` bootstrap for registries with DNS issues
- Available domains show a direct **Register →** link to Namecheap
- ↺ Retry per-row for any that time out

### AI Brain Visualisations
- **Loss curve** — canvas-rendered training loss over 80 steps, with gradient fill and step labels
- **Next-character probabilities** — animated bar chart of the top 10 softmax outputs from the last generation step
- **Attention heatmap** — n×n grid showing causal masking and attention weights between characters
- All visualisations update on every generation run

### Educational Section — How LLMs Work
Five chapters covering the full transformer stack, each with collapsible concept cards, code snippets, diagrams, and plain-English analogies:

| Chapter | Topics |
|---|---|
| **Foundation** | Tokenisation, BPE, embeddings, positional encoding, RoPE |
| **Transformer** | Attention (Q/K/V), multi-head attention, MLP, residual stream, RMSNorm |
| **Training** | Next-token prediction, cross-entropy loss, backpropagation, autograd, Adam optimiser |
| **Generation** | Temperature, top-k/nucleus sampling, autoregressive loop, KV cache |
| **Scale** | This model vs GPT-2/3/4 side-by-side, FLOPs, batching, quantisation, Flash Attention |

---

## How the model works

NameForge is a faithful JavaScript port of the microgpt algorithm by [@karpathy](https://gist.github.com/karpathy/8627fe009c40f57531cb18360106ce95).

### Architecture

```
Input tokens → Token embedding (16d) + Positional embedding (16d)
             → RMSNorm
             → Multi-head self-attention (4 heads × 4d = 16d)
                 ├── Causal masking (no future token leakage)
                 └── Scaled dot-product attention per head
             → Residual connection
             → RMSNorm
             → MLP (16d → 64d ReLU → 16d)
             → Residual connection
             → Linear projection → Logits (vocab size = 27)
```

### Key parameters

| Parameter | Value | GPT-3 equivalent |
|---|---|---|
| Embedding dimension | 16 | 12,288 |
| Attention heads | 4 | 96 |
| Layers | 1 | 96 |
| Context window | 16 chars | 2,048 tokens |
| Vocabulary | 27 tokens | ~50,000 tokens |
| Parameters | ~4,000 | 175 billion |
| Training steps | 80 | ~300 billion token passes |

### Tokeniser

Character-level: `a–z` mapped to indices `0–25`, plus a `BOS` (beginning-of-sequence) token at index `26`. The BOS token doubles as the stop signal — generation halts when the model predicts it again.

### Training

`model.train(80, onStep)` runs 80 gradient steps using a cosine-decayed learning rate starting at `0.014`. Each step samples one name from the corpus, runs a forward pass, computes cross-entropy loss against the next-character targets, and nudges the output projection weights in the direction that increases the probability of the correct character. The loss curve animates live.

### Generation

Autoregressive sampling: a seed sequence (extracted from your product description keywords, or a vibe-specific prefix) is fed in, and the model appends one character at a time until it predicts BOS or reaches the max length. Temperature is set per vibe mode — `0.45` for Clean (conservative) up to `0.85` for Playful (creative).

---

## Domain checking

No proxy. No API key. No account.

RDAP (Registration Data Access Protocol) is the modern, structured replacement for WHOIS, maintained by ICANN. Every domain registry is required to run a public RDAP server. A `404` response means no registration record exists — the domain is available. A `200` means a record was found — taken.

### Registry endpoints used

| TLD | Authority | RDAP server |
|---|---|---|
| `.com` | Verisign | `rdap.verisign.com/com/v1/domain/` |
| `.net` | Verisign | `rdap.verisign.com/net/v1/domain/` |
| `.io` | NIC.io | `rdap.nic.io/domain/` |
| `.co` | NIC.co | `rdap.nic.co/domain/` |
| `.app` | Google Registry | `rdap.nic.google/rdap/domain/` |
| `.dev` | Google Registry | `rdap.nic.google/rdap/domain/` |
| `.ai` | NIC.ai | `rdap.nic.ai/domain/` |
| `.org` | Public Interest Registry | `rdap.publicinterestregistry.org/rdap/domain/` |

If a direct registry endpoint fails (DNS resolution error, timeout), the lookup falls back to `rdap.org`, which bootstraps to the correct authoritative server automatically.

---

## Technical details

### Zero dependencies

No npm. No bundler. No build step. No runtime frameworks. No external scripts beyond Google Fonts. The entire application — transformer engine, visualisations, domain checker, educational content — ships in a single `.html` file.

### Browser requirements

Any modern browser (Chrome 90+, Firefox 88+, Safari 15+). Requires:
- `fetch` API with `AbortController`
- `Canvas 2D` for loss curve rendering
- `CSS backdrop-filter` for the sticky header blur (degrades gracefully)

### Font stack

- **Playfair Display** — headings and name display (serif, editorial)
- **DM Sans** — body text (variable, humanist sans)
- **Fragment Mono** — labels, code, metadata (monospace)

All loaded from Google Fonts. The app degrades to system fonts if offline.

---

## Deployment

### GitHub Pages (recommended)

```bash
git clone https://github.com/MoraRosa/NameForge
cd NameForge
# Rename to index.html if deploying to root
# Or keep as-is and access at /NameForge/
```

Push to any public repo, go to **Settings → Pages → Deploy from branch → main**, and it's live. No build step.

### Local

```bash
# Just open the file
open index.html
# or
python3 -m http.server 8080  # then visit localhost:8080
```

> **Note:** Domain checking requires network access to the RDAP registry servers. It works correctly when served over HTTP/HTTPS. Some browsers restrict `fetch` from `file://` origins — use a local server if domain checks don't resolve.

---

## Inspiration & credits

- **[microgpt](https://gist.github.com/karpathy/8627fe009c40f57531cb18360106ce95)** by [@karpathy](https://github.com/karpathy) — the 200-line pure Python GPT that this transformer is ported from
- **[Neural Networks: Zero to Hero](https://www.youtube.com/playlist?list=PLAqhIrjkxbuWI23v9cThsA9GvCAUhRvKZ)** by Andrej Karpathy — the lecture series the educational section is structured around
- **[The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)** by Jay Alammar — visual reference for the attention diagrams
- **[Attention Is All You Need](https://arxiv.org/abs/1706.03762)** — Vaswani et al., 2017

---

## Project structure

```
NameForge/
└── index.html        # The entire application — ~2,600 lines
    ├── <style>       # CSS custom properties, layout, components (~700 lines)
    ├── <body>        # HTML structure — hero, form, output, learn section
    └── <script>      # JS — transformer, domain checker, visualisations (~900 lines)
```

---

## License

MIT — do whatever you want with it.

---

*Built by [MoraRosa](https://github.com/MoraRosa)*
