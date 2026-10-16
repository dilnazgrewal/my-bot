# Kapoor Knitwear — website chatbot

A sales-desk chatbot and one-page website for **Kapoor Knitwear**, a fictional family-run knitwear maker and wholesaler in Ludhiana, Punjab. Built as a classroom project.

The chatbot answers questions about products and wholesale terms, and collects wholesale enquiries. When a buyer confirms, it sends a structured enquiry to the owner (Google Sheet / Telegram).

> This is a classroom demo for a **fictional business**. All contact details are placeholders and nothing here is a real offer.

## What's in here

| File | What it is |
|------|------------|
| `worker.js` | The chatbot backend. Runs as a Cloudflare Worker. Holds the business facts and the bot's rules, and keeps the AI API key out of the browser. |
| `site/index.html` | The one-page website, with its own CSS. No frameworks, no build step. |
| `site/chat-widget.js` | The chat bubble, added to the page before `</body>`. Unchanged from the starter kit. |
| `run-tests.js` | Sends the 15 test questions to the live bot and saves the replies. |
| `test-results.md` | The results of that test run: 15 / 15. |

## Run the website locally

You need [Python](https://www.python.org/) (or Node — see below). From this folder:

```bash
python -m http.server 8787 --directory site
```

Then open **http://localhost:8787**. Press `Ctrl + C` to stop.

With Node instead, any static server works, for example:

```bash
npx --yes serve site -l 8787
```

## Deploy the chatbot

The website is just static files, but the chat needs `worker.js` running somewhere.

1. Create a Worker at [dash.cloudflare.com](https://dash.cloudflare.com) and paste in `worker.js`.
2. In **Settings → Variables and Secrets**, add:

   | Name | Value | Type |
   |------|-------|------|
   | `LLM_BASE_URL` | e.g. `https://api.groq.com/openai/v1` | Text |
   | `LLM_MODEL` | e.g. `openai/gpt-oss-120b` | Text |
   | `LLM_API_KEY` | your own API key | **Secret** |
   | `ALLOWED_ORIGINS` | the address your site is served from | Text |

   Optional, to send enquiries to the owner: `LEAD_WEBHOOK_URL` (Google Apps Script / Zapier / Make / n8n), or `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`.

3. Copy the Worker's address and put it in `site/index.html`, in `window.SalesDeskConfig`, with `/chat` on the end:

   ```js
   endpoint: "https://YOUR-WORKER.workers.dev/chat",
   ```

### About `ALLOWED_ORIGINS`

This is the list of websites allowed to use the chat backend from a browser. If it doesn't match where the page is served from, the chat replies *"This website is not allowed to use this chat backend."*

- While testing locally: `*`
- Once published: your real address, for example `https://your-site.pages.dev`

**The API key is never in this repository and never in the website.** It lives only in the Worker's settings, as a Secret.

## Run the tests

With the bot deployed and its address set in `run-tests.js`:

```bash
node run-tests.js
```

It asks all 15 questions as separate chats, then holds one full enquiry conversation, waiting between requests because free AI plans allow only a few a minute. Re-run just some of them with `node run-tests.js 7,12,15`. The replies are written to `test-output.json`; the marked-up results are in `test-results.md`.

## Editing the bot

`worker.js` is in four sections:

1. **Settings** — limits. Leave alone.
2. **Business knowledge** — the facts the bot is allowed to state. If a fact isn't here, the bot won't claim it.
3. **Bot rules** — what it does, what it refuses, and the format of the enquiry it sends to the owner.
4. **Code** — the plumbing. Leave alone.

To change what the bot knows or how it behaves, edit sections 2 and 3 only. The enquiry field names in section 3 must stay exactly as they are, because section 4 and the owner's Google Sheet depend on them.
