# Deploying Mira on Vercel (no Cloudflare)

This version runs entirely on Vercel — the web page **and** the proxy that holds your
API key live in the same project. Your key stays server-side; students never see it.

> The old `mira-proxy-worker.js` (Cloudflare) is no longer used. You can delete it.

## What goes in your GitHub repo

```
your-repo/
├── index.html          ← the app (rename reflection-companion.html to index.html)
└── api/
    └── chat.js         ← the serverless proxy
```

Vercel automatically turns any file in the `/api` folder into a serverless function,
so `api/chat.js` becomes the live endpoint `https://YOUR-SITE.vercel.app/api/chat`.
The app is already set to call `/api/chat` on its own domain — students enter nothing.

## Why students saw "แอปยังไม่ได้ตั้งค่า"

The deployed page had an empty `CONFIG`, so it had no server to talk to. That's now fixed:
the app points to `/api/chat` and includes the class passcode. You just need to add the
`api/chat.js` function and set the environment variables below.

---

## Step 1 — Add the files to your repo

1. Rename **`reflection-companion.html`** to **`index.html`** and keep it at the repo root
   (so students get a clean URL like `your-site.vercel.app`). *(If you prefer to keep the
   name, the URL becomes `your-site.vercel.app/reflection-companion.html` — also fine.)*
2. Create a folder named **`api`** and put **`chat.js`** inside it.
3. Commit and push to GitHub. Vercel will start a deploy automatically.

## Step 2 — Set environment variables in Vercel

In Vercel: **your project → Settings → Environment Variables.** Add:

| Name | Value | Required |
|------|-------|----------|
| `CLASS_PASSCODE` | `refTes6789` | Yes — must match the passcode baked into the app |
| `OPENAI_API_KEY` | your OpenAI key (`sk-…`) | If using OpenAI |
| `ANTHROPIC_API_KEY` | your Anthropic key (`sk-ant-…`) | If using Claude |
| `DEFAULT_PROVIDER` | `openai` or `anthropic` | Optional (defaults to openai) |
| `OPENAI_MODEL` / `ANTHROPIC_MODEL` | model override | Optional |

Set them for the **Production** environment (and Preview if you use it).

## Step 3 — Redeploy

Environment variables only take effect on a new deploy. In Vercel:
**Deployments → ⋯ on the latest one → Redeploy** (or just push any commit).

## Step 4 — Edit the safety resources (do this before sharing widely)

Open `api/chat.js`, find the line:

```
[INSERT YOUR INSTITUTION'S RESOURCES HERE ...]
```

Replace it with your university's counseling/advisor contacts, verify the Thai hotlines
(1323 / 1669 / 191), commit, and let Vercel redeploy.

## Step 5 — Test

1. Open your Vercel URL in a normal browser tab (no `#teacher`).
2. Send a message. You should get a *question* back (never advice). Try the ไทย toggle.
3. If you get "Invalid class passcode," the `CLASS_PASSCODE` env var doesn't match
   `refTes6789` in the app — make them identical and redeploy.

---

## Security & cost notes

- **The class passcode is visible in the page source** (it's baked into `index.html`).
  Real protection is your API key, which lives only in Vercel env vars and never reaches
  the browser. The passcode just discourages casual outsiders.
- For more protection: keep the GitHub repo **private**, rotate `CLASS_PASSCODE` each term
  (update it in both the env var and `CONFIG.PASSCODE` in the HTML), and set a **monthly
  spend limit** in your OpenAI/Anthropic billing dashboard.
- To change the passcode later: edit `CONFIG.PASSCODE` in `index.html` **and**
  `CLASS_PASSCODE` in Vercel, then redeploy. They must always match.

## Teacher mode

Add `#teacher` to your URL (e.g. `your-site.vercel.app/#teacher`) to reveal the settings
gear and the system prompt. Students using the plain URL never see these.
