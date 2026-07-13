# API Chat Console

A single-file, self-contained BYOK (Bring Your Own Key) chat client for any OpenAI-compatible LLM API. No backend, no build step, no account — it's one `.html` file that runs entirely in your browser.

## What it is

This started as a simple request: a GUI to manage API keys and chat with them. It grew into a full local ChatGPT-style client that talks to whichever inference provider you want, with your own keys, entirely on your own machine.

There's no server component. Opening the file in a browser is the entire deployment. It works offline except for the actual API calls to whichever model provider you've configured.

## Core features

**Multi-provider chat**
- Works with any API that implements the OpenAI `/chat/completions` schema: OpenAI, Groq, Cerebras, NVIDIA Build, and Google's Gemini via its OpenAI-compatibility endpoint (`https://generativelanguage.googleapis.com/v1beta/openai`).
- Add unlimited named endpoints (base URL + key + default model), switch between them from a dropdown.
- Streaming responses.
- Added Export, Import button in settings panel for quick export and import of Search keys, chats, currentChatId, app prefs and user profiles

**Chat management**
- Sidebar with full chat history, one conversation per entry.
- New chat, delete chat, and inline rename (✎ icon).
- Per-message edit (editing a user message truncates everything after it and regenerates; editing an assistant message just updates the text) and delete.
- "Redo" / regenerate on the last assistant response.
- Per-chat system prompt field.

**Attachments**
- Image upload — sent in OpenAI's vision content-block format (`image_url` with a base64 data URL), so it works with any vision-capable model on a compatible endpoint.
- Text file upload (`.txt`, `.md`, `.csv`, `.json`, code files) — content is read and appended into the message as context.

**Formatting**
- Lightweight, dependency-free markdown renderer: bold, italics, inline code, fenced code blocks, bullet and numbered lists, headers.
- Rough token-count estimate per message and in the input box (character-count heuristic, not a real tokenizer — good enough for a sanity check, not exact).

**Web search (provider-agnostic)**
- A "Web Search" toggle that doesn't depend on the LLM provider having its own search tool (most don't — OpenRouter and Gemini are the exceptions).
- Instead, when enabled, the app itself calls **Firecrawl's search API** first; if that errors, times out, or has no key set, it automatically falls back to **Tavily's search API**.
- Results get formatted and injected as a system message ahead of your query, so the model answers with live web context regardless of which inference provider is doing the actual generation.

**Customize (global user profile)**
- A ChatGPT/Claude-style custom-instructions panel: full name, preferred nickname, a short freeform bio, and a preferred response language.
- This gets merged into the system message on every chat automatically, layered on top of any per-chat system prompt — so you don't have to re-explain who you are every conversation.

**Appearance**
- Accent color picker and a choice of three font stacks (sans / serif / monospace), saved locally.

## Storage & privacy model

This is the part that I built this project for, and it's deliberately split into three tiers depending on sensitivity:

| Data | Where it lives | Persists across refresh? |
|---|---|---|
| Provider endpoint keys (OpenAI, Groq, Cerebras, etc.) | JS memory only | **No** — cleared on refresh/close, by design |
| Firecrawl / Tavily search keys | `localStorage` | Yes |
| Chat history (messages, attachments, titles) | `IndexedDB` | Yes |
| Appearance & Customize profile | `localStorage` | Yes |

The reasoning: your primary inference keys are the highest-value secret in this tool, so they were deliberately kept out of any persistent store — you paste them once per session and they're gone the moment the tab closes. The search keys and personalization data were explicitly persistent across all sessions for convenience, so those are saved. Chat history moved from `localStorage` to `IndexedDB` specifically to lift the ~5-10MB browser storage cap (which image-heavy conversations could hit) up to whatever your disk allows.

None of this data goes to any third party except the exact API endpoints you configure. There is no analytics, no telemetry, no external server involved in running the app itself.

## Known limitations

- **CORS**: some self-hosted inference servers (e.g. a default Ollama install) don't send CORS headers, so a direct browser `fetch()` to them will be blocked. Hosted providers built for client-side use (OpenAI, Groq, Cerebras, NVIDIA Build, Gemini's OpenAI-compat endpoint) work fine.
- **Token counts are estimates**, not exact — real tokenization varies by model/tokenizer, this uses a `chars ÷ 4` heuristic.
- **No true model-side function calling** — the web search feature works by pre-fetching results and injecting them as context, not by giving the model a callable tool. This is intentional (it's what makes it work across every provider uniformly), but it means the model can't decide *when* to search — the toggle either searches before every message or it doesn't.
- **`localStorage`-backed data is scoped to this exact file/origin** — if you move or rename the `.html` file, or open it in a different browser, saved search keys / profile / appearance won't carry over (chat history in IndexedDB behaves the same way).
- Not a "vault" — nothing here is encrypted at rest. It's appropriately scoped for personal convenience, not for storing anything you'd consider highly sensitive.

## Tech

Vanilla HTML/CSS/JS. No frameworks, no build tooling, no npm install. Chat persistence via the native `IndexedDB` API, everything else via `localStorage`. Single file, opens directly in any modern browser (tested against Firefox-based browsers like Floorp).

## Provider quick-reference

| Provider | Base URL |
|---|---|
| OpenAI | `https://api.openai.com/v1` |
| Groq | `https://api.groq.com/openai/v1` |
| Cerebras | `https://api.cerebras.ai/v1` |
| NVIDIA Build | `https://integrate.api.nvidia.com/v1` |
| Gemini (OpenAI-compat) | `https://generativelanguage.googleapis.com/v1beta/openai` |
