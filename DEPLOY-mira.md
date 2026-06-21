# Deploying Mira — the reflection companion for your students

You have three files:

- **`reflection-companion.html`** — the app students open on a web browser.
- **`mira-proxy-worker.js`** — the Cloudflare Worker that holds your API key and checks the class passcode. Students never see your key.
- **`DEPLOY-mira.md`** — this guide.

The flow: student's browser → your Worker (checks passcode, adds your secret key) → OpenAI/Anthropic → back. Setup takes about 15 minutes and runs on Cloudflare's free tier.

---

## Part 1 — Deploy the Worker (the proxy)

1. Create a free account at **dash.cloudflare.com**.
2. In the left menu: **Workers & Pages** → **Create** → **Create Worker**. Give it a name like `mira-proxy`. Click **Deploy** (it deploys a placeholder).
3. Click **Edit code**. Delete everything in the editor, then paste the entire contents of **`mira-proxy-worker.js`**. Click **Deploy**.
4. Note your Worker URL — it looks like `https://mira-proxy.YOUR-NAME.workers.dev`. You'll need it in Part 2.

### Add your secrets and settings

Go to your Worker → **Settings** → **Variables and Secrets**. Add the following.

Add as **Secret** (encrypted — use these for anything sensitive):

| Name | Value |
|------|-------|
| `CLASS_PASSCODE` | A passcode you invent, e.g. `reflect2026`. Students will type this once. |
| `OPENAI_API_KEY` | Your OpenAI key (from platform.openai.com → API keys). Add only if you use OpenAI. |
| `ANTHROPIC_API_KEY` | Your Anthropic key (from console.anthropic.com). Add only if you use Claude. |

Add as **Variable** (plain text — optional):

| Name | Value |
|------|-------|
| `DEFAULT_PROVIDER` | `openai` or `anthropic` — which one to use when the app doesn't specify. |
| `OPENAI_MODEL` | Override the default (`gpt-4o-mini`). |
| `ANTHROPIC_MODEL` | Override the default (`claude-3-5-haiku-latest`). |
| `ALLOWED_ORIGIN` | The site you'll host the HTML on, e.g. `https://yourname.github.io`. Leave unset to allow any origin. |

Click **Deploy** again after adding variables so they take effect.

> You only need a key for the provider(s) you'll actually use. Setting both lets you switch.

---

## Part 2 — Prepare and host the app (the HTML)

### Bake in the settings (recommended — students do nothing)

Open **`reflection-companion.html`** in a text editor. Near the top of the `<script>` you'll find:

```js
const CONFIG = {
  SERVER_URL: "",   // e.g. "https://mira-proxy.your-name.workers.dev"
  PASSCODE:   "",   // e.g. "reflect2026"
  PROVIDER:   ""    // "", "openai", or "anthropic"
};
```

Fill in your Worker URL and passcode, e.g.:

```js
const CONFIG = {
  SERVER_URL: "https://mira-proxy.your-name.workers.dev",
  PASSCODE:   "reflect2026",
  PROVIDER:   ""
};
```

Now students just open the page and start. (If you leave these blank, students will instead be asked to enter the server address + passcode the first time — fine if you'd rather share the passcode separately for a little extra control.)

### What students see vs. what you see

Once you've baked in the server URL and passcode, the **settings gear is hidden from students entirely** — they have nothing sensitive to open. The passcode field and the "View the system prompt" section appear **only in teacher mode**.

To open teacher mode (to check settings or read the prompt), add `#teacher` to the end of the URL — e.g. `https://yoursite.com/index.html#teacher`. Bookmark that version for yourself; share the plain URL (without `#teacher`) with students.

> **A note on real secrecy:** hiding the UI stops students from *casually* seeing the passcode, but anyone who opens the browser's "View Source" can still find anything baked into the HTML. The passcode is only a soft gate — your actual protection is the API key, which lives encrypted in the Worker and never reaches the browser. If you want the passcode itself kept off students' machines, leave `CONFIG.PASSCODE` blank and give students the passcode through a separate channel (an LMS announcement); they type it once and it's stored only in their own browser.

### Edit the safety resources (important — do this before sharing)

Open **`mira-proxy-worker.js`** and find the line:

```
[INSERT YOUR INSTITUTION'S RESOURCES HERE ...]
```

Replace it with your university's counseling center, student-affairs contact, or advisor info, and verify the Thai hotline numbers (1323 / 1669 / 191). Re-paste the file into the Worker and **Deploy** again.

### Put the HTML somewhere students can open it

Any static host works. Easiest free options:

- **GitHub Pages** — create a repo, upload `reflection-companion.html` (rename to `index.html`), enable Pages.
- **Cloudflare Pages**, **Netlify Drop** (netlify.com/drop — drag the file in), or your university's LMS file hosting.
- Or just email students the file — it runs locally by double-clicking, as long as `CONFIG` is filled in.

Then share the link (and the passcode, if you didn't bake it in).

---

## Part 3 — Test it

1. Open the app link.
2. Type a message like "I have a decision I'm stuck on."
3. You should get a question back (never advice). Try the ไทย toggle and write in Thai — it should reply in Thai.
4. To confirm the gate works: temporarily change the passcode in the app's Settings to something wrong — you should get an "Invalid passcode" error.

---

## Keeping costs down

- The default models (`gpt-4o-mini`, Claude Haiku) are inexpensive — typically a fraction of a cent per exchange.
- The Worker trims long conversations and rejects oversized ones automatically.
- Rotate the passcode each term, and set a **monthly spend limit** in your OpenAI/Anthropic billing dashboard as a safety net.
- If the passcode leaks, change `CLASS_PASSCODE` in the Worker and redeploy — old links stop working immediately.

---

## Reusing the prompt as a ChatGPT custom GPT (optional)

The full system prompt lives inside `mira-proxy-worker.js` (the `SYSTEM_PROMPT` constant). You can copy that text into OpenAI's GPT builder ("Instructions" field) to make a shareable custom GPT instead — set Name to "Mira", turn off web browsing and image generation for a focused tool, and add the same conversation starters.
