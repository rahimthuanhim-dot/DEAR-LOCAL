# Dear Local

A static, single-page local news site for Kahilipara, Guwahati, and Assam — community-submitted updates via a Google Form, no backend server required.

## Stack

- Plain HTML + [Tailwind CSS](https://tailwindcss.com) (via CDN)
- Vanilla JavaScript (no build step)
- Google Forms + Google Sheets + Apps Script as the content backend
- Deployed on [Vercel](https://vercel.com)

## Project structure

```
.
├── index.html          # the site
├── config.js            # your Apps Script + Form URLs (not committed with real values — see below)
├── apps-script.gs       # backend script — paste this into Apps Script, not part of the deployed site
├── assets/
│   └── dearlocal-logo.png
├── robots.txt
└── sitemap.xml
```

## How content publishing works

1. Someone fills out the Google Form (fields: `Section`, `Category`, `Headline`, `Summary`, `Source Link`, `Submitted by`).
2. The response lands as a new row in the linked Google Sheet.
3. `apps-script.gs`, deployed as a Web App, reads that Sheet and returns approved rows as JSON, split into `local` and `assam` arrays based on the `Section` value.
4. `index.html` fetches that JSON on page load and renders cards into the two grids.

**There is no manual approval step.** Every valid submission publishes immediately, the same as the school notice board pattern. Keep the Form link limited to people you trust, or add an approval gate back in if this ever needs to be public-facing (see `apps-script.gs` for where the filter would go).

## Setup from scratch

1. **Create the Form** with these exact field names (exact matters — the backend matches Sheet columns by name):
   - `Section` — dropdown: `Local`, `Assam`
   - `Category` — short answer
   - `Headline` — short answer
   - `Summary` — short answer
   - `Source Link` — short answer (URL)
   - `Submitted by` — short answer, optional
2. **Link the Form to a Sheet** (Responses tab → green Sheets icon).
3. **Copy the Sheet ID** from its URL: `https://docs.google.com/spreadsheets/d/THIS_PART/edit`
4. **Open Extensions → Apps Script** from inside that Sheet, paste in `apps-script.gs`, and set `SHEET_ID` at the top to the ID from step 3.
5. **Deploy → New deployment → Web app.** Execute as **Me**, access **Anyone**. Copy the `.../exec` URL.
6. **Fill in `config.js`:**
   ```js
   window.CONFIG = {
     NEWS_SCRIPT_URL: "https://script.google.com/macros/s/XXXXX/exec",
     SUBMIT_FORM_URL: "https://forms.gle/XXXXX",
   };
   ```
7. Test locally with a real local server (VS Code's **Live Server** extension, or `npx serve .`) — opening `index.html` directly via `file://` will silently break the fetch call, since browsers block `fetch()` from `file://` origins.

## Deploying updates

Any time you edit `apps-script.gs` in the Apps Script editor, saving alone does **not** push it live — go to **Deploy → Manage deployments → edit (pencil icon) → New version → Deploy**.

## Known limitations

- No approval/moderation on submissions — anyone with the Form link can publish instantly.
- No malformed-URL validation on `Source Link` beyond what the Form itself enforces; garbage input just renders as plain text instead of a link.
- Before going live, update the placeholder domain (`dearlocal.vercel.app`) in `index.html`'s canonical/Open Graph tags, `robots.txt`, and `sitemap.xml` to your real deployed URL.
