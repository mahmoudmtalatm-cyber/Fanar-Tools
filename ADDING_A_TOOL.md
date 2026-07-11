# Adding a new tool to Fanar Tools

This project is a hub-and-tools setup for a Telegram bot with multiple mini apps.
Read this before starting a new chat about adding tool #3, #4, etc. — paste this
whole file in, or just say "read ADDING_A_TOOL.md" if the file is attached.

## Project structure

```
fanar-tools/
├── index.html                        ← the hub ("Fanar Tools" menu). This is the
│                                        ONE url your bot's persistent Menu Button
│                                        should point to.
├── assets/
│   └── logo.png                      ← shared lighthouse logo, used by hub + every tool
└── tools/
    └── <tool-name>/
        └── index.html                ← each tool is fully self-contained, own <style>
                                          and <script>, no shared JS/CSS files
```

Every tool is a single standalone HTML file. Nothing is shared between tools except
the logo image and the visual style (see "Design system" below). This is
intentional — it means one tool can never break another, and you can hand a whole
tool's file to me in a fresh chat with zero other context needed.

## Step-by-step: adding a new tool

1. **Create the folder and file:**
   `tools/<tool-name>/index.html` (kebab-case name, e.g. `tools/flashcard-maker/index.html`)

2. **Reference the shared logo with a relative path going up two levels:**
   `../../assets/logo.png`
   (Same pattern the slide-extractor tool uses — copy its `<head>` and topbar
   markup as a starting template, then swap the body/screens.)

3. **Add a "back to hub" link** in the topbar, same as the existing tool:
   ```html
   <div class="back" id="home-link">...</div>
   <script>
     document.getElementById('home-link').addEventListener('click', () => {
       window.location.href = '../../index.html';
     });
   </script>
   ```

4. **Register the tool in the hub.** Open `index.html` at the project root and
   find the `TOOLS` array near the bottom of its `<script>` block:
   ```js
   const TOOLS = [
     {
       name: 'Slide Extractor',
       desc: 'Turn a lecture video into a clean PDF of its slides',
       path: 'tools/slide-extractor/index.html',
       enabled: true,
       icon: '<rect x="4" y="3" width="16" height="18" rx="2"/><path d="M8 8h8M8 12h8M8 16h5"/>'
     },
     // add your new tool object here, above the "More tools soon" placeholder
   ];
   ```
   Add a new object with the new tool's `name`, `desc`, `path` (relative to
   `index.html`, so `tools/<tool-name>/index.html`), `enabled: true`, and an
   `icon` (inline SVG path/shape data — grab one from lucide.dev and paste just
   the inner shape tags).

   That's the *only* edit needed in the hub file. Nothing else changes.

5. **Delete or leave the "More tools soon" placeholder** — it's just a disabled
   card showing there's room to grow; doesn't need to be removed.

That's the whole integration surface — one new folder, one new array entry.

## Design system to reuse (keep tools visually consistent)

Every tool should keep these CSS custom properties at the top of its `<style>`
block (copy verbatim from any existing tool):

```css
--navy-deep:#0B2340;  --navy:#12365C;   --blue:#2E6FB0;   --blue-soft:#DCEAF7;
--gold:#F0A93E;       --gold-soft:#FCE3B2; --paper:#F6FAFD;
--ink:#10233B;        --ink-mute:#5C7690;  --line:#E1EBF4;  --radius:18px;
```

Fonts: `Space Grotesk` (headings/wordmark), `Inter` (body), `IBM Plex Mono`
(logs, counters, timestamps) — same Google Fonts `<link>` tag as the existing
tool.

Keep the `.app` max-width:430px mobile-first shell, the `.topbar` logo+wordmark
pattern, and `.btn-primary` (gold gradient) for the main call-to-action button.

## Telegram wiring every tool file should include

```html
<script src="https://telegram.org/js/telegram-web-app.js"></script>
```
```js
const tg = window.Telegram ? window.Telegram.WebApp : null;
if (tg) {
  tg.ready();
  tg.expand();
  try { tg.setHeaderColor('#0B2340'); tg.setBackgroundColor('#F6FAFD'); } catch(e) {}
}
```

## Bot-side note (not part of this repo)

- **Menu Button** (the icon beside the message box) can only point to ONE url —
  keep it pointed at the hub's `index.html`.
- You can *also* deep-link straight to any tool from an inline keyboard button
  with its own `web_app: {url: '.../tools/<tool-name>/index.html'}` — useful for
  a `/command` shortcut into a specific tool without going through the hub.
- "No server deployment" only rules out custom backend logic. Telegram still
  requires an HTTPS url, so this repo needs static hosting (GitHub Pages,
  Cloudflare Pages, Netlify, etc.) — no backend code required, just file hosting.
