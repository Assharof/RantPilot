# CLAUDE.md — RantPilot

## PROJECT CONTEXT (added for this repo)
- Local project: `C:\laragon\www\rantpilot`. Laravel 12 + Breeze (Blade + Tailwind) + MySQL database `rantpilot` (root, no password).
- Dev workflow: terminal 1 `php artisan serve --port=8001` (http://127.0.0.1:8001, because port 8000 is used by YOUWATCH), terminal 2 `npm run dev` (Vite).
- Reference screenshots: `docs/reference/` (indexed in `docs/reference/README.md`). Look at them before building any screen.
- The owner is a beginner. After each step, explain in simple words what you built, which files changed, and how to check it in the browser. Then STOP and wait.

Rebuild the existing web app **Stitchable** (`stitchable.99dfy.com`, reference `https://1min.site/start`) as closely as possible from the reference screenshots. Match layout, spacing, typography, colors, components, states and behavior. **Do not invent features not shown.**

## RULES (apply to every step)
- Pixel-faithful to the screenshots. If unclear, **ask before guessing**.
- Use exact text from screenshots (copy strings below verbatim).
- Keyboard-accessible, semantic HTML, proper `<label>`s, visible focus states.
- Mobile responsive.
- Anything marked **ASSUMED** is an inference, not visible in a screenshot. Anything marked **ASK** must be confirmed before building.
- Build one step at a time (see Section 7) and **wait for confirmation** before the next step.

## 0. WHAT THE PRODUCT IS
Turn any YouTube video into a hosted mini-course page. The user pastes a YouTube URL and its transcript, AI splits the video into short clips (sections) with titles and generates marketing headline options, the user adds an optional timed call-to-action button, then saves to the cloud and shares a link.

Tagline under logo: **"Your Personal RANT Assistant"**
Marketing promise: "7 clicks", "60 seconds", "0 tech skills".

**Screens identified from screenshots:**
1. Editor (`New` / builder) — Add Headline + Video preview + Timed Button settings + right sidebar
2. Editor — Bulk Import panel (empty, loading, and AI-result states)
3. Published / preview course page (viewer)
4. Marketing landing page `1min.site` (copy supplied as text only — **no visual screenshot; ASK**)

Not shown in any screenshot (do NOT build detail, only stub routes): `My Stitchables`, `Transcribe`, `Run RANT`, login/signup, pricing/billing.

---

## 1. GLOBAL DESIGN SYSTEM
All hex values are **ESTIMATED from screenshots** — verify with an eyedropper on the original before finalizing.

### Color tokens
| Token | Value | Where used |
|---|---|---|
| `--color-bg` | `#FFFFFF` | page background |
| `--color-surface` | `#FFFFFF` | cards, panels |
| `--color-surface-muted` | `#F9FAFB` (ASSUMED) | live preview area, inputs bg |
| `--color-border` | `#E5E7EB` | card/input borders |
| `--color-text` | `#111827` | headings, titles |
| `--color-text-muted` | `#6B7280` | sub-headlines, helper text, timestamps |
| `--color-primary` | `#4F46E5` (indigo, approx.) | logo text, active clip border, number badges, "Bulk Import" tab, "Add Timed Button", slider fill/thumb, radio |
| `--color-primary-soft` | `#A5B4FC` approx. | disabled "Import Clips", "Generating Timestamps..." button |
| `--color-cta-bg` | `#3B82F6` | timed button default background (visible: `#3b82f6`) |
| `--color-cta-border` | `#1D4ED8` | timed button default border (visible: `#1d4ed8`) |
| `--color-cta-text` | `#FFFFFF` | timed button default text (visible: `#ffffff`) |
| `--color-badge-yellow` | `#FBBF24` approx. | "S.A.V.O.R" badge |
| `--color-badge-text` | `#1F2937` approx. | text on yellow badge |

Dark mode exists (a **Dark** toggle is in the header). Dark palette is **not shown → ASK**.

### Typography
- Font family: clean sans-serif (Inter-like, **ASSUMED**). The *user-styled CTA button* uses a selectable font family (default **Verdana**).
- Logo domain text: bold, ~16px, indigo.
- Logo tagline: ~10–11px, muted gray.
- Page headline (viewer): bold, ~28px, centered, dark.
- Sub-headline (viewer): ~16px, regular, muted, centered.
- Card section title ("Add a Headline", "Bulk Import Clips"): semibold ~16–17px.
- Field labels ("Headline", "Alignment", "Width"): ~12px, medium/semibold.
- Helper text / timestamps: ~10–11px muted.
- Clip title: ~12px semibold. Clip time: ~10px muted.
- Timed button default: 22px, Bold, Verdana.

### Radius, shadow, spacing
- Cards/panels: radius ~12px, 1px border `--color-border`, very subtle shadow (almost flat).
- Inputs/selects/textareas: radius ~6–8px, 1px border.
- Buttons: radius ~6–8px (timed-button preview ~6px).
- Clip rows: radius ~8px, 1px border; active row = 2px indigo border.
- Number badge: 24–28px circle, indigo fill, white numeral.
- Spacing scale (ASSUMED 4px base): 4 / 8 / 12 / 16 / 24 / 32.
- Editor layout: main column ≈ 580px wide, right sidebar ≈ 210px wide, gap ≈ 40px, content centered ≈ 960px wide.

### Components
- **Primary button**: indigo fill, white text, ~12px semibold, icon + label (e.g. upload icon + "Bulk Import").
- **Secondary button**: white fill, 1px border, dark text, icon + label (e.g. "+ Single Clip", "Save to Cloud", "Export Json", "Import Json").
- **Header nav button**: white/light with border, ~12px text, small icon where shown.
- **Input / textarea**: white, 1px border, placeholder in light gray; textarea has a resize handle bottom-right.
- **Select**: same as input with a chevron at right.
- **Slider (range)**: filled indigo track to thumb, light-gray remainder, white circular thumb with indigo outline.
- **Color field**: small rounded color swatch (≈26×18px) + hex text input.
- **Radio card**: bordered rounded card, radio at top-left, bold title, muted description.
- **Progress bar**: thin gray track with percent label right ("0%").
- **Icons**: outline/line icons, ~14–16px (lucide-style, ASSUMED): play-in-circle, lightbulb, cloud, download, upload, pencil, trash, external-link, drag-handle (6 dots), chevron, moon, user, list, film.

---

## 2. APP SHELL
Single top header, white, bottom border, content centered (~960px). No sidebar.

**Left:** logo — blue/indigo rounded-square icon (film/clapper glyph) + two-line text:
- `stitchable.99dfy.com` (bold, indigo)
- `Your Personal RANT Assistant` (small, muted)

**Right, in order:**
1. `New` (button)
2. `My Stitchables` (list icon + label)
3. `Transcribe`
4. `Run RANT`
5. `Dark` (moon icon) — theme toggle
6. User menu: user icon + email (`engr.mbari@gmail.com` in screenshots) + chevron-down → dropdown (**contents ASSUMED, ASK**: likely account / sign out)

**States:** active nav state **not visible → ASK**. Collapsed/mobile state **not shown → ASSUMED**: nav collapses into a hamburger menu below ~768px; logo stays; sidebar column stacks below the main column.

**Header is hidden in the published viewer** (screenshot 6 shows only the course page inside the browser) — viewer is a public page with no app header (ASSUMED).

**Navigation targets (ASSUMED, ASK):** New → blank editor; My Stitchables → list of saved projects; Transcribe → transcript tool; Run RANT → unknown feature ("RANT" meaning not explained in screenshots).

---

## 3. PAGE-BY-PAGE SPEC

### 3.1 Editor — route `/` or `/new` (ASSUMED)
**Purpose:** build one Stitchable (headline, clip sequence, timed CTA).

**Layout:** two columns under the header.
- **Left (main, ~580px):** Video preview card → "Add a Headline" card → Timed Button settings.
- **Right (sidebar, ~210px):** Video Sequence card (clips + actions), mode tabs, Bulk Import panel.

#### Left column
**A. Video preview card** (bordered, rounded, ~120px tall in empty state)
- Empty state: centered play-in-circle icon (gray), title **"No Video Selected"** (bold), sub **"Add some video clips to start your sequence"**.
- Filled state: 16:9 embedded YouTube player of the active clip; above it the section row (see viewer 3.3). Plays only the clip's start→end range (ASSUMED).

**B. "Add a Headline" card**
- Title: **Add a Headline**; right-aligned button with lightbulb icon **Use Template**.
- Label **Headline** → textarea, placeholder: `How [Target Audience] [Achieve Desired Result] In [Time Frame] Without [Common Obstacle]`
- **Font Size: 28px** → slider (live label updates, e.g. "Font Size: 28px"). Range min/max **ASK** (thumb sits ~60% along at 28px).
- Label **Sub-headline (Optional)** → textarea, placeholder: `The proven [Number]-step system that [Specific Benefit] - even if [Common Objection]`
- **Font Size: 16px** → slider (thumb ~25% along at 16px).
- **Use Template** behavior: fills the headline + sub-headline fields with the placeholder pattern (ASSUMED).

**C. Timed Button settings** (scrolled further down; has no visible card title → ASK)
- **Delay:** three number inputs labelled **Hours**, **Minutes**, **Seconds** (values 0 / 0 / 3). Helper: **Total delay: 3 seconds** (updates live).
- **Alignment:** select, value **Center**.
- **Width:** text input `800px`. **Height:** text input `80px` (side by side).
- **Background Color:** swatch + hex `#3b82f6`. **Border Color:** swatch + hex `#1d4ed8` (side by side).
- **Text Color:** swatch + hex `#ffffff` (full width).
- **Font Size:** `22px`. **Font Weight:** select `Bold` (side by side).
- **Font Family:** select `Verdana` (full width).
- **Live Preview:** gray-tinted preview box showing the button with label **Click Here to Secure Your Spot Now →** styled with the values above.
- Full-width primary button: **+ Add Timed Button**.
- **Button label field and URL field are not visible → ASK** (the label text appears only in the preview; link target unknown).

#### Right column
**D. Video Sequence card** — header: indigo dot icon + **Video Sequence (5 clips)** (count updates live).
- Buttons: **Save to Cloud** (cloud icon, full width), **Export Json** (download icon), **Import Json** (upload icon) — last two side by side.
- Clip list (sample data, exact):
  | # | Title | Range |
  |---|---|---|
  | 1 | Start Lean | 00:00:00 → 00:01:20 |
  | 2 | Attract Crowds | 00:05:01 → 00:06:28 |
  | 3 | Visual Systems | 00:10:00 → 00:11:28 |
  | 4 | Optimize Pricing | 00:15:00 → 00:16:30 |
  | 5 | Run With Heart | 00:20:00 → 00:21:30 |
- Each row: drag handle (left), indigo numbered circle, title (with small external-link icon in viewer / play icon in editor), time range, and in editor: **play** (filled indigo square when active, outline otherwise), **edit** (pencil), **delete** (trash).
- Active row = 2px indigo border.
- Reordering by drag handle (implied by handle; behavior ASSUMED).

**E. Mode tabs** (below the clip list): **+ Single Clip** (secondary) | **Bulk Import** (primary/filled when active).
- **Single Clip form is not shown → ASK.** (Fields likely URL, title, start, end — ASSUMED.)

**F. Bulk Import panel** — card title with icon: **Bulk Import Clips**
1. **Video URL** — input, placeholder `https://youtube.com/watch?v=... or direct vid` (text is cut off in screenshot; "direct vid[eo]" ASSUMED).
2. **Copy the full transcript from https://tactiq.io** — textarea (small, scrollable) containing a pasted Tactiq transcript, e.g. lines like:
   `00:29:55.520 225. Like and subscribe and we'll see you next` / `00:29:58.000 …`
3. Primary button (loading state): spinner + **Generating Timestamps...** (disabled, light indigo), with text **Processing transcript...** and a progress bar at **0%** (percent right-aligned).
4. **Time Ranges (one per line)** — textarea, placeholder:
   ```
   https://youtube.com/watch?v=VIDEO_ID
   Introduction
   00:01:30 → 00:02:45
   Main Content
   00:03:15, 00:04:30
   ```
5. Helper text: `Format: YouTube URL (optional), Title (optional) followed by hh:mm:ss → hh:mm:ss or hh:mm:ss, hh:mm:ss or hh:mm:ss - hh:mm:ss`
6. Button **Import Clips** (upload icon; disabled/light indigo until time ranges exist).

**G. AI result — Marketing Headlines** (appears in sidebar after generation)
- Collapsible header: flag icon + **Marketing Headlines - 3 Options Available** + chevron (up when open).
- Three radio cards (option 1 pre-selected):
  1. **How First-Time Entrepreneurs Built a $22K/Week Food Truck in Just 7 Months Without a Culinary Background** — *The proven 5-step system that turned late-night soul food into a booming business — even if you've never worked in a kitchen*
  2. **Copy My Exact Food Truck Playbook That Generated $22,000 a Week** — *Step-by-step blueprint that launched a viral soul food brand — no guesswork, no culinary degree*
  3. **Give Me 7 Months and I'll Show You How to Build a 6-Figure Food Truck** — *Fast, predictable results — even if you're starting from zero with no restaurant background*
- Below option 3 a raw AI-output fragment is visible: `## RECOMMENDED: Use Option 1 – … points:`. **Treat as leaked markdown; ASK whether to show or strip.**
- Selecting a radio fills Headline + Sub-headline in the editor (ASSUMED).

**Loading / empty / error states**
- Loading: shown above (Generating Timestamps…, 0% progress).
- Empty: "No Video Selected" card; clip list empty-state **not shown → ASK**.
- Error / validation states **not shown → ASK**; minimum ASSUMED: inline red helper text under invalid fields.

**Section label on each clip (AI-generated):** a yellow badge **S.A.V.O.R** appears next to the clip name (see 3.3). Meaning/source of "SAVOR" **unknown → ASK** (looks like a framework label/acronym whose letters map to clips, e.g. S = Start Lean).

### 3.2 My Stitchables / Transcribe / Run RANT — **ASK**
No screenshots. Create empty routes with page title only until provided.

### 3.3 Published course page (viewer) — route `/s/:slug` (ASSUMED)
**Purpose:** public mini-course page the buyer/lead sees.

**Layout:** two columns, no app header.
- **Left (≈ 590px):**
  1. Headline — bold centered: *How First-Time Entrepreneurs Built a $22K/Week Food Truck in Just 7 Months Without a Culinary Background*
  2. Sub-headline — muted centered: *The proven 5-step system that turned late-night soul food into a booming business — even if you've never worked in a kitchen*
  3. Section row: left = `1. Start Lean` + yellow **S.A.V.O.R** badge; right = **Clip 1 of 5** (muted).
  4. 16:9 YouTube embed (shown: title "This $22K/Week Food Truck Started Just 7 Months Ago", big red play button, Share, "Watch on YouTube").
  5. Timed CTA button (appears after the configured delay): **Click Here to Secure Your Spot Now →**, blue `#3b82f6`, `#1d4ed8` border, white bold text, centered.
  6. Playback control row (centered, small icon buttons): **previous**, **play** (filled indigo), **stop** (square), **next**.
- **Right (≈ 270px):** card **Video Sequence (5 clips)** with the 5 clips (same data as above), each: indigo number circle, title + small external-link icon, time range. Active clip has 2px indigo border. Clicking a clip loads it.

**Interactions**
- Click clip → loads that clip range in player, updates "Clip N of 5" and section label.
- Next/prev change clip; play/stop control playback.
- CTA button is hidden until delay elapses, then appears (in screenshot 2 it is visible at ~2:00 of the tutorial, i.e. after delay).
- Auto-advance to next clip when a clip ends: **ASK**.
- External-link icon next to clip title: opens the source video at that timestamp (ASSUMED).
- Mobile: sidebar stacks below the player (ASSUMED).

### 3.4 Marketing landing page — `1min.site` (copy provided as text; **no screenshots → ASK for visuals before building**)
Sections in order, **exact copy from the brief:**
- **Nav:** How It Works · Features · Pricing · FAQ · **Get Started**
- **Hero:** badge `⚡ Now in public beta — Join 2,400+ creators`; H1 `Stop Creating Content.` / `Start Publishing Mini Courses In 60 Seconds.`; sub `Turn any YouTube video into your own hosted course site. No filming. No editing. No tech skills. Just 7 clicks and you're live.`; CTAs `Start Building Free →` and `Watch Demo`; stats `7 Clicks to launch` · `60s Average build time` · `0 Tech skills required`
- **The Problem** — `You've been told to "create content" a thousand times.` / `Here's why you still haven't launched anything.` — 6 cards: 📹 Recording is exhausting · ✂️ Editing takes forever · 🔧 Tech stack overwhelm · 📝 Copywriting paralysis · 🎨 Design decisions drain you · 📁 The "Someday" folder grows (each with the body text supplied in the brief)
- **The Solution** — `What if the content already existed?` — 3 cards: 🎯 AI-Powered Structuring · ⚡ Instant Page Generation · ☁️ Cloud Hosting Included
- **How It Works** — `Seven clicks. That's it.` / `Here's exactly what happens when you use 1min.site` — steps 01–07 (Copy any YouTube video URL → Paste it into 1min.site → Pull the transcript automatically → AI analyzes and structures the content → Generate your hosted course page → Add your call-to-action → Save and go live)
- **Features** — `Everything you need. Nothing you don't.` — 6 cards: 📊 Smart Content Compression · ✍️ AI-Generated Headlines · 🎨 Professional Templates · 💳 Built-In Monetization · 🌐 Instant Cloud Hosting · 📱 Mobile-Optimized
- **Use Cases** — `Four ways to monetize` — 1 Sell as a paid product · 2 Use as a lead magnet · 3 Create a content library · 4 Bonus for existing products
- **Pricing, FAQ, footer: nav links exist but no content provided → ASK.**
- Note: the landing page claims "payment processor", "lead capture form" and "email opt-in"; the app screenshots show **only a timed CTA button**. Build only the button; treat the rest as marketing copy (**ASK**).

---

## 4. DATA MODEL (all inferred — every item is ASSUMED)
- **User** — id, email (`engr.mbari@gmail.com` visible), theme preference (light/dark).
- **Stitchable** (a.k.a. mini-course page) — id, owner_id, slug/url, source_video_url, headline, headline_font_size (default 28), subheadline, subheadline_font_size (default 16), selected_headline_option, status (draft/published), created_at, updated_at.
- **Clip** — id, stitchable_id, position (order), title, video_url (optional per clip), start_time, end_time (hh:mm:ss), section_label (e.g. "S.A.V.O.R" badge — meaning unknown).
- **TimedButton** — id, stitchable_id, delay_hours, delay_minutes, delay_seconds, alignment (default Center), width (800px), height (80px), bg_color (#3b82f6), border_color (#1d4ed8), text_color (#ffffff), font_size (22px), font_weight (Bold), font_family (Verdana), label ("Click Here to Secure Your Spot Now →"), link_url (**unknown field, ASK**).
- **HeadlineOption** — id, stitchable_id, title, subtitle, is_selected (3 per generation).
- **Transcript** — id, stitchable_id, raw_text (Tactiq format `hh:mm:ss.mmm text`), source ("tactiq.io").
- **Export/Import JSON** — serialization of Stitchable + Clips + TimedButton (schema ASSUMED).
- Relations: User 1—N Stitchable; Stitchable 1—N Clip (ordered); Stitchable 1—1 (or 1—N, **ASK**) TimedButton; Stitchable 1—N HeadlineOption; Stitchable 1—1 Transcript.

---

## 5. USER FLOWS
**Core flow (marketed as 7 clicks; the narration in the walkthrough maps clicks 4, 7 explicitly):**
1. Copy a YouTube URL (outside the app).
2. Paste into **Video URL** (Bulk Import).
3. Paste the transcript copied from tactiq.io into the transcript box.
4. Click the generate button → **Generating Timestamps…** with progress; AI returns clips + 3 marketing headlines (walkthrough: "wait for AI to work for a couple of seconds").
5. Review the clips list and pick a headline option (default option 1).
6. Optional: open timed button settings → set delay, style → **+ Add Timed Button**.
7. Click **Save to Cloud** → page is live; share link.

**Edit flow:** Edit pencil on a clip → change title/range (form **not shown, ASK**); trash to delete (confirm dialog **ASK**); drag to reorder; play icon to preview.
**Export/Import:** Export Json downloads a file; Import Json uploads one to restore.
**Auth/signup/login:** not shown. Only a logged-in user email is visible → **ASK** (provider, flows).
**Settings/billing:** not shown; landing page mentions "Pricing" → **ASK**.

---

## 6. LOGIC & INTEGRATIONS
| Capability | What the UI implies | Status |
|---|---|---|
| YouTube embed/playback | Embed the video, restrict to start→end per clip, controls prev/play/stop/next | Implied |
| Transcript import | User manually pastes transcript from **tactiq.io**; Transcribe nav item likely automates this | Manual paste visible; automation **ASSUMED, ask me** |
| AI clip generation | Reads transcript → returns clip titles + hh:mm:ss ranges + SAVOR-style labels | **ASSUMED, ask me** which AI provider/prompt |
| AI headlines | 3 options (headline + sub-headline), one flagged "RECOMMENDED" | Implied; prompt **ASSUMED, ask me** |
| Time-range parsing | Accept `hh:mm:ss → hh:mm:ss`, `hh:mm:ss, hh:mm:ss`, `hh:mm:ss - hh:mm:ss`, optional URL and title lines | Format text visible |
| Cloud save/hosting | "Save to Cloud", public shareable URL, no domain setup | Implied; hosting/slug scheme **ASK** |
| Timed CTA | Appears after Hours/Minutes/Seconds delay | Implied |
| Payments / lead capture / email opt-in | Mentioned only in landing copy | **Not in app screenshots — ASK** |
| Run RANT | Unknown feature | **ASK** |
| Dark mode | Header toggle | Palette **ASK** |

---

## 7. BUILD ORDER (one step at a time — wait for my confirmation after each)
1. **Design tokens + shell** — tokens from Section 1, header (logo, nav, Dark toggle, user menu), responsive behavior, focus states.
2. **Pages with mock data** — Editor (all cards, sliders, timed-button settings, live preview), Bulk Import states (empty / loading 0% / result with 3 headline options), Viewer page with the 5 sample clips, stub routes for My Stitchables / Transcribe / Run RANT. Landing page only after I supply visuals.
3. **Real data layer** — Stitchable/Clip/TimedButton persistence, Save to Cloud, Export/Import Json, reorder/edit/delete, public viewer URL.
4. **Auth** — only after I confirm the flows (nothing visible in screenshots).
5. **Integrations** — YouTube player range playback, transcript parsing, AI clip + headline generation, timed CTA behavior.
6. **Polish + responsive + accessibility** — keyboard nav (drag-reorder alternative controls), ARIA for radio cards/collapsible/progress, contrast check, dark mode, mobile stacking.

---

## 8. OPEN QUESTIONS (answer before the relevant step)
1. Visual references for the landing page, My Stitchables, Transcribe, Run RANT, login, pricing/FAQ?
2. Dark-mode palette and user-menu dropdown contents?
3. Single Clip form and Edit Clip form fields?
4. What does "S.A.V.O.R" stand for and how is it assigned to clips?
5. Timed button: label/URL fields, and does the settings block have a title?
6. Slider min/max for the two headline font sizes?
7. Show or strip the `## RECOMMENDED: Use Option 1 …` text in the headline panel?
8. Does the viewer auto-advance clips? Where does Save to Cloud publish (URL scheme)?
9. Exact hex values — OK to eyedrop from the original app at stitchable.99dfy.com?
