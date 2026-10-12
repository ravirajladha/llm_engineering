# Week 1 Notes

## Day 1 - First LLM call: website summarizer
- Keep API keys in `.env` and load them with `load_dotenv(override=True)`. Never hard-code keys.
- The `openai` Python package is a thin wrapper around an HTTP call. It contains no model code.
- **System prompt** = the task, persona and tone. **User prompt** = the actual input.
- Messages are a list of dicts:
  ```python
  [{"role": "system", "content": "..."},
   {"role": "user", "content": "..."}]
  ```
- The core call (you'll write this all course):
  ```python
  response = client.chat.completions.create(model=MODEL, messages=messages)
  response.choices[0].message.content
  ```
- `scraper.fetch_website_contents(url)` is a simple requests + BeautifulSoup scraper. It fails on JS-rendered sites (you'd need Selenium/Playwright for those).

## Day 2 - Chat Completions API, endpoints, Ollama
- "Chat Completions" means: here is a conversation, predict what comes next.
- You can call the endpoint with plain `requests.post(...)`, and the client library just does that for you.
- **OpenAI-compatible endpoints**: most providers copied OpenAI's API, so the same `OpenAI` client works everywhere. Only `base_url`, `api_key` and `model` change:

  | Provider | base_url |
  |---|---|
  | Anthropic (Claude) | `https://api.anthropic.com/v1/` |
  | Google (Gemini) | `https://generativelanguage.googleapis.com/v1beta/openai/` |
  | Ollama (local) | `http://localhost:11434/v1` (any api_key) |
- **Ollama** runs open-source models locally (`ollama pull llama3.2:1b`, `ollama serve`).
  - Pros: free, and your data never leaves your machine.
  - Cons: much weaker than frontier models.
- Homework: `day2_homework.ipynb` is the day 1 summarizer running on Ollama, compared against Claude.

## Day 3 - Frontier models
- A survey of the frontier models (GPT, Claude, Gemini, Grok, DeepSeek, ...) and their strengths. No notebook.

## Day 4 - Tokens and the "illusion of memory"
- Models read **tokens**, not words (about 4 characters, or 0.75 words, per token in English). Use `tiktoken` to inspect OpenAI tokenization.
- Pricing and context windows are measured in tokens.
- **LLM calls are stateless.** The model remembers nothing between calls.
- "Memory" is a trick: you resend the whole conversation (system, user, assistant, user, ...) on every call. Longer chats therefore cost more tokens.

## Day 5 - Brochure generator (multi-step business solution)
- Chain multiple LLM calls:
  1. Give the LLM all the links on a site and ask it to pick the relevant ones (About, Careers, ...).
  2. Scrape those pages, then ask the LLM to write a company brochure.
- **Structured output**: ask for JSON with `response_format={"type": "json_object"}`, then `json.loads(...)` the result.
- **One-shot / few-shot prompting**: include an example of the desired output in the prompt.
- **Streaming**: pass `stream=True`, then loop over the chunks and append `chunk.choices[0].delta.content or ""`. In Jupyter, use `display(..., display_id=True)` with `update_display(...)` for the typewriter effect.

## Week 1 exercise (`week1 EXERCISE.ipynb`)
A technical Q&A tutor: asks the same question to Claude (`claude-haiku-4-5`, streamed) and to local Llama 3.2 1B via Ollama, using a single `stream_answer(client, model)` function. This shows the whole week in one place: prompts, messages, OpenAI-compatible clients, streaming, and frontier vs. local models.

## Key takeaways
1. One API shape (Chat Completions) works with almost every provider.
2. System prompt + user prompt + message list is the basic building block.
3. LLMs are stateless, so context = everything you send.
4. Real apps chain several LLM calls and use structured (JSON) outputs between steps.
