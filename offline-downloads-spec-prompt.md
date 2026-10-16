# Prompt: spec + plan for the "Grandma Downloads" web app

Paste everything below the line into a fresh session (strongest model, web search enabled), inside the new, empty GitHub repo.

---

You are a senior engineer and product designer. Produce a **SPEC** and an **IMPLEMENTATION PLAN** (no application code yet) for a small personal web app. Write both as files in this repo (`SPEC.md`, `PLAN.md`). Ask me up to 5 clarifying questions only if truly blocking; otherwise state your assumptions explicitly in the spec.

## 1. Context

My grandmother has an iPhone and a Google TV stick running Stremio with Real-Debrid (Torrentio addon, I believe). She often goes far into her large garden where there is no mobile signal, and she wants to watch her shows offline on her iPhone. She normally watches on a big TV, so smaller files are fine on a phone screen and save space.

She is decently tech-comfortable but wants as few steps as possible. She is English-speaking, small text is fine, and she uses subtitles.

**The manual flow that already works for her today:**
real-debrid.com -> Downloads (Safari on iPhone) -> file saves to the Files app -> plays offline in VLC or Infuse.

I want to turn that into a one-tap experience.

## 2. Goal

A private, password-protected web app, installable on her iPhone home screen (PWA), that:

1. Lists what is already in my Real-Debrid account (recent downloads and torrents), in a clean, large-tap-target, grandma-friendly layout, grouped by show/season where it can be inferred from file names, with posters if available without extra paid APIs.
2. Lets her download an item with one big button. The file goes **straight from Real-Debrid to her phone** (saved to the Files app), then plays offline in VLC/Infuse.
3. Helps her keep phone storage under control as far as iOS allows.
4. Helps her pick a smaller version of an item.
5. Nudges her to download at home on Wi-Fi before heading into the garden. It must never need a signal in the garden.

## 3. Facts and decisions already made

- **Users:** my grandmother and me. A single shared password. She is fully self-serve.
- **Real-Debrid account:** mine, shared with her. Addon: Torrentio (confirm only if it matters).
- **Usage:** she follows 1-2 shows at a time. Her iPhone has about 10 GB free, maybe more.
- **Scope of v1:** only list what is already in the Real-Debrid account. No search, no indexing, no sourcing of content in the app. I am responsible for making sure she downloads only content she has the right to use.
- **Hosting:** Vercel (free tier), domain `amaurymarque.com` registered on Porkbun (use a subdomain, e.g. `tv.amaurymarque.com`), code in a GitHub repo that deploys from `main`. Do **not** compare hosting options.
- **Budget:** as close to free as possible. I will maintain it myself.
- **Smaller files, v1:** no server-side re-encoding (Vercel cannot run ffmpeg well). Instead show every file in a torrent with its size, recommend the smallest good 1080p/720p one, and show something like "about N episodes fit in your free space".
- **List hygiene:** a "Done / Watched" button hides an item from her main list. Items older than 30 days are hidden behind a "Show older" link. Nothing is ever deleted from Real-Debrid.
- **Language:** English only.

## 4. Hard constraints and iOS limits (design around these honestly; do not promise what iOS cannot do)

- **No native iOS app.** PWA / Safari only (add-to-home-screen, manifest, service worker only if actually useful).
- A web page **cannot see or delete files in the Files app** and **cannot read true free storage**. Propose realistic mitigations:
  - `navigator.storage.estimate()` as a rough hint, labelled as an estimate;
  - show the file size before downloading;
  - a "downloaded on this phone" list tracked in the app (localStorage/IndexedDB) with a manual "I deleted this" toggle;
  - warnings when the total tracked downloads approach a user-set storage budget (default 10 GB);
  - short, clear instructions with screenshots for deleting files in the Files app.
- **iOS Safari downloads** can pause when the phone locks or Safari goes to the background. Design UX for this: a "keep the screen on until it finishes" hint, a clear progress/"did it finish?" check, and "tap to try again".
- **Large files must not pass through Vercel serverless functions** (time and size limits). The backend only fetches metadata and unrestricts links; the browser downloads directly from Real-Debrid.
- **Secrets:** the Real-Debrid API token must NEVER reach the browser or the repo. It lives in a Vercel environment variable. All Real-Debrid calls go through the backend.
- **Auth:** one shared password, strong and simple, with HTTPS only, secure httpOnly session cookie, rate limiting / lockout on login attempts, `noindex`, and no public pages beyond the login.
- **Verify against current docs, do not guess:**
  - Real-Debrid API docs (api.real-debrid.com): endpoints for listing downloads, torrents, torrent info, unrestricting links, and rate limits;
  - whether download links are IP-locked, and what that means for "backend unrestricts, phone downloads";
  - whether the `/streaming/transcode` endpoints give downloadable lower-quality files or are stream-only, and what that implies for a future v2;
  - iOS Safari/PWA limits (storage, downloads, install behaviour) for the current iOS version.
  Cite the sources you rely on.

## 5. Deliverables

**SPEC.md**
1. Product summary and non-goals.
2. User stories for my grandmother and for me (admin tasks: set token, rotate password).
3. Screens, described at wireframe level in text: login, home list (shows, seasons, items), item detail with file picker and sizes, "my phone" storage/downloaded list, help page. Include large-text styling rules, tap targets of at least 48 px, high contrast, and plain-language copy.
4. Error states and copy (token invalid, Real-Debrid down, download stopped, not enough space estimated, wrong password).
5. Architecture: recommended stack (default suggestion: Next.js on Vercel, but justify), API routes, data model (what is stored where), auth design, secret handling, PWA manifest and service worker scope, caching rules.
6. How file names are parsed into show/season/episode, and the fallback when parsing fails.
7. Security and privacy checklist.

**PLAN.md**
1. Phased plan: an MVP buildable in a weekend, then v2 ideas (poster art, re-encoding, a "request" button, push reminders).
2. Tasks with acceptance criteria for every phase.
3. Test plan, including testing on a real iPhone in airplane mode, plus unit tests for the filename parser and API route tests with a mocked Real-Debrid.
4. Step-by-step setup: create the Vercel project, connect the GitHub repo, add the Porkbun DNS record for the subdomain, set environment variables, add to the iPhone home screen.
5. Risks and open questions.
6. A short note for my grandmother: 5 simple steps in plain language ("download at home first", "open the app", "tap the big button", "open it in VLC", "delete when you've finished").

Keep it concise and decision-oriented. Prefer the simplest thing that works. When finished, summarise the key decisions and any questions for me.
