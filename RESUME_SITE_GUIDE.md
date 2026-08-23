# RESUME_SITE_GUIDE.md — subproject guide and log

*Load this before touching anything in `Resume_Site/`. Sub-project of `Career/`; the parent entry
file is `../CAREER_REFERENCE.md`, which holds the fact base, the GATEKEEPER and the career
decisions. This file holds everything specific to the site.*

**Last updated: 2026-08-09.**

---

## 1. What this folder is

**`Resume_Site/` is a byte-for-byte mirror of the GitHub repo `jzesbaugh/resume`.** Filenames must
match the repo exactly, because they are uploaded as-is.

```
Resume_Site/
├── index.html              ← the page. Canonical copy lives HERE.
├── README.md               ← repo documentation, public-facing
├── og.png                  ← 1200x630 social preview. MUST ship with index.html.
├── svg-flier-front.png     ← Selected work image, added rev 3o. 700px wide, resized from source.
├── svg-flier-back.png      ← Selected work image, added rev 3o. 700px wide, resized from source.
├── wotw-trailer-thumb.png  ← added rev 3p. Trailer still for War of the Worlds, self-hosted so
│                              the video link is a static image + play glyph, not a YouTube embed.
└── RESUME_SITE_GUIDE.md    ← this file. NOT uploaded. Internal only.
```

**⚠ Source files for the flier images live in `../../Vegan Store/` (`SVG_Flier_4x6_FRONT.png` /
`_BACK.png`), NOT here.** The copies in this folder are resized (700px wide) for web weight and are
the ones that ship. Do not confuse the two locations, and re-copy + re-resize if the source design
changes.

**Live:** <https://jzesbaugh.github.io/resume/>
**Repo:** <https://github.com/jzesbaugh/resume>

> **⚠ `RESUME_SITE_GUIDE.md` must never be uploaded to the repo.** It contains internal reasoning,
> weighting rules and correction history that has no business on a public page. Only `index.html`
> and `README.md` go up.

### Why the folder exists

Until 2026-08-07 the canonical file was `Career_Resumes/Resume_Interactive_WIP.html` and the repo
held a copy called `index.html`. **Two files, two names, guaranteed drift.** Consolidating to a
mirror folder means one local copy, one remote copy, same filename, and a SHA check that actually
proves they match.

---

## 2. Deployment — the procedure, and the two traps

### To deploy

1. Edit the files here
2. **⚠ Update the `Last updated` line in the footer of `index.html`.** It is hand-maintained, so
   it goes stale silently and a stale date on a live page is worse than no date. **Do this before
   computing hashes**, or the SHA you record will not match what you upload.
3. Compute local blob SHAs (see below)
4. Upload **`index.html`, `README.md`, `og.png`, `svg-flier-front.png`, `svg-flier-back.png`,
   `wotw-trailer-thumb.png` and `svg-guide-screenshot.png`** (the last four added rev 3o/3p/3s)
   to `github.com/jzesbaugh/resume` via **Add file → Upload files**. Same filenames replace the
   existing ones
5. Commit to `main`
6. Verify by SHA before telling anyone it is live

> **⚠ `og.png` must be in the repo root.** The OG tags point at an absolute URL,
> `https://jzesbaugh.github.io/resume/og.png`. Relative paths do not resolve for social scrapers.
> **If the image is missing, every shared link silently renders as bare text** — which is the bug
> that prompted rev 3e in the first place.

### After deploying, force the social caches to re-read

LinkedIn, Facebook and Slack cache OG data per-URL, sometimes for weeks, and they cache the
*absence* of an image too. **Changing the tags does not update an already-scraped link.**

| Platform | Tool |
|---|---|
| LinkedIn | Post Inspector — `linkedin.com/post-inspector/` |
| Facebook / Meta | Sharing Debugger — `developers.facebook.com/tools/debug/` |
| Twitter / X | Card Validator |
| Slack | Post the link in a private channel to see the current unfurl |

Paste the URL and re-scrape. **Do this once after the first deploy with `og.png`**, or the old
imageless preview persists.

### To verify

```bash
git hash-object index.html README.md
```

Then compare against:

```
https://api.github.com/repos/jzesbaugh/resume/contents/
```

which returns `sha` and `size` per file. **Match = deployed.** This also detects local/remote drift.

### 🪤 Trap 1 — `raw.githubusercontent.com` is CDN-cached

It served a **stale rev 1 copy minutes after rev 3b was committed**, which produced a confident and
completely wrong "the upload didn't land" conclusion. Do not use it to check deployment.

### 🪤 Trap 2 — the rendered page is cached too, at several layers

Fetching `jzesbaugh.github.io/resume/` returned a version **older than either of two uploads made
that evening.** Browser cache, Pages cache and CDN all sit in the way. Hard-refresh is `Cmd+Shift+R`.

### 🪪 Trap 4 — `/contents/` can return a stale 200, which is worse than an empty one

**2026-08-09:** `/contents/` — the endpoint this section designates as reliable — returned a
well-formed 200 describing **rev 3b** (`1b4aa9b3…` / 28,744 bytes) when the repo actually held
rev 3k. Two calls returned byte-identical payloads. Taken alone it would have produced a confident
"the last several revisions never deployed" verdict.

**What caught it:** rendering `jzesbaugh.github.io/resume/` and looking for a feature that only
exists in the newer revision — the Options bar, the twelve linked answers, the OG tags. That is an
independent path, and it is cheap.

> **Revised rule: never report a NEGATIVE deploy result from a single endpoint.** "Reliable"
> was earned by this endpoint failing *loudly* while others failed silently — it had never been
> observed failing *quietly*, and now it has. A hash match still proves deployment; a hash
> mismatch no longer disproves it on its own. Candidate #48 in `../../AI settings/SETTINGS_LOG.md`.

### 🪤 Trap 3 — the GitHub API rate-limits unauthenticated requests

After a handful of calls, `/branches/main`, `/commits` and `/git/trees/` all started returning
**empty bodies rather than errors**, which is easy to misread as "the file is not there."
The `/contents/` endpoint was the most reliable. If several endpoints go quiet at once, suspect the
rate limit rather than the repo.

> **The lesson, stated once:** this deployment was called wrong in *both* directions in a single
> session — a stale page read as a failed upload, then the reverse. **Only the blob SHA is
> evidence.** Everything else is a guess with a confident tone.

---

## 2c. Printing — and why it matters more than it sounds

**Printing is the PDF story.** Rather than maintaining a separate PDF that drifts out of sync, the
page prints properly and the OS dialog offers *Save as PDF*. One document, no drift.

> **⚠ The bug this fixed, worth remembering.** Until rev 3h, printing captured **only the single
> answer that happened to be open on screen** — a printed copy was silently missing eleven of
> twelve. Nothing surfaced the problem; it just quietly produced an incomplete resume. **A hidden
> `#printall` div is now built from `DATA` at load and revealed only under `@media print`.**

**If you edit `DATA`, the print block updates automatically.** It is generated from the same
array, so the two cannot diverge. **Do not hand-author a second copy of the answers for print.**

Check after any structural change: print preview should show the header, the lede, all twelve
questions and answers with their sources, then experience, education and selected work — and none
of the buttons.

---

## 2b. Theming

**Three themes: `dark` (default), `light`, `terminal`.** Set by `data-theme` on `<html>`. The
button **cycles** and always advertises the *next* theme, so a click has a predictable result.

**Dark is the default for everyone.** This is deliberate and overrides the reader's OS preference
on first visit.

### Terminal theme — and the objection that was withdrawn

Phosphor green on near-black, monospace. **I argued against building this** on the grounds that it
reads as trying too hard and fights the build-neutral decision. **The second half of that objection
was wrong:** it is opt-in, so nobody sees it unless they choose it, and a theme a reader selects
cannot position Jesse by default. Recorded because the reasoning matters more than the verdict.

- The green is **deliberately not** the classic `#33FF33`. That vibrates against black and is
  genuinely unreadable at paragraph length. Body text is `#8FEF9F` at about **14:1** against the
  background; the dimmest tier, `#3E8A54`, still clears **4.6:1**.
- **Monospace runs wide.** `.name` and `.lede` step down a size under this theme, with a further
  reduction under 620px, or the name overflows on a phone.

> **⚠ Print resets every theme to ink-on-paper** via a `--var` override inside `@media print`.
> Without it, dark mode printed grey body text on white and terminal printed **green**. That bug
> predated the terminal theme and had been shipping since dark mode existed.

> **The trade, stated honestly.** Respecting `prefers-color-scheme` is the better-behaved default,
> and forcing a theme on someone who chose light is the kind of thing that reads as a website
> having opinions about your eyes. **The visible toggle is what makes it acceptable** — a reader
> who wants light is one click away, and their choice sticks. **If the toggle is ever removed,
> the default must go back to respecting the system setting.**

- Applied by a small inline script in `<head>`, before the stylesheet, so there is no flash of the
  wrong theme
- Persisted in `localStorage` under `jz-theme`, `try/catch` wrapped because it throws in some
  privacy modes; the fallback is simply dark
- `<html>` also carries the attribute statically, so **no-JS readers get dark rather than a broken
  half-state**
- The button is labelled with the **action**, not the current state — "Light" means "switch to
  light." A toggle labelled with its current state is ambiguous about what clicking it does
- `meta theme-color` is `#0F0F0F` to match the default
- Print always forces light, and the toggle is hidden in print

---

## 3. Hard rules for this page

These are gates. They were each learned by getting it wrong first.

| # | Rule | Why |
|---|---|---|
| **0** | **⭐ Check before you generate** | The parent rule. **Eight times in one session an answer was produced from what was in context when a one-call check was available** — the question set, the accent colour, the weighting, two attributions, the deploy status twice, a file error. **The tell is noticing you could write something plausible right now.** Route it: conventions → web search; world facts → search or tool; **Jesse's reasoning, history or ranking → ask him**; what a source says → re-read it; deployed state → query it by hash. Candidate #46. |
| 1 | **Check 0b — weight by consequence, not by documentation volume** | *"A handful of dirt is not worth more than one grain of gold."* Tier order: real-world scale → shipped and in use → published → academic. See `../CAREER_REFERENCE.md`. |
| 2 | **Do not re-compress the answers** | This format has no length constraint. An earlier draft wrote them as resume bullets and silently dropped the most persuasive material in the record. 3–5 short paragraphs, split on `\n\n`. |
| 3 | **Do not open on the career-change question** | It is a challenge, not an opener. Leading with it reads defensive before a reader has any reason to doubt. Open on "tell me about yourself" — asked in 90%+ of interviews. |
| 4 | **Questions come from research, not invention** | The original set was made up to suit the available material and missed the single most-asked question entirely. Wording is taken from what hiring managers actually ask. |
| 5 | **Phase 1 is a written interview, not a chatbot** | "Cannot fabricate" is *how it works*, not a feature to advertise. Disclaimers denying hallucination, and decline answers demonstrating refusal, belong to phase 2. See §5. |
| 6 | **Every "I identified / I decided" must be quotable from Jesse** | Two claims reached the live page that the record did not support. Structured analysis manufactures plausible occupants for empty cells. See §6. |
| **1b** | **⭐ Stay build-neutral — describe the record, not a target role** | **A1 is undecided by choice** until a real posting exists. *"IT, compliance and operations"* is a positioning claim, not a fact, and it narrows him before a reader has said what the job is. Titles, meta, JSON-LD and `og.png` describe **what he has done**. Where "compliance and operations" appears inside the lede or the About-me answer it is a **list of functions he covered** — history, not positioning — and that is fine. **Do not re-slant without a posting to slant toward.** |
| **1c** | **⭐ The AI answer stays in plain language. Do not re-promote the methodology.** | The rule-testing work (versioned instruction set, control-tested, rules that lost to a no-rules baseline) is real. **⚠ PREMISE CHANGED 2026-08-09:** this rule was justified by the methodology being *"unpublished and uninspectable."* **It is now published** — the repo carries three phases, `PROTOCOL_V2.md`, `MECHANISM_TAXONOMY.md`, two case studies and the control-arm design, so a reader can open it. **The rule survives anyway, on the other half of its reasoning:** Check 0 ranks by *adoption*, not by page count, and the repo is still Tier 3 — published, not adopted by anyone else — which puts it below the shipped app either way. It also still has more documentation behind it than anything else on the page, which is exactly the condition that inflated the capstone. **Lead on the app and this page. No jargon.** The Selected work entry was updated in rev 3l to describe the control arm and its unflattering result; the AI *answer* was deliberately left alone. |
| **8** | **⚠ Watch the "I would rather X than Y" tic — and audit for repetition across ALL answers, not within one** | **Found 2026-08-09 on a full read of all twelve.** The construction appeared **seven times**: *rather find out early than…*, *rather be concrete than…*, *rather tell you than let you discover…*, *rather name both than have you find them*, *rather say that plainly than have you find it*, *rather be the person who reports his own outage than the person who…*, *rather be busy and learning than comfortable*. Two consecutive answers — **What I want** and **What I'd bring** — opened with near-identical *"Three things, and I would rather…"* sentences. **Cut to two deliberate uses**, both as closers. **The mechanism worth naming:** each answer was written and reviewed on its own, so every instance read fine in isolation and nothing surfaced the pattern. Jesse hit three of them one at a time as a reader, which is how a reader encounters them and how the drafting process never did. **A twelve-answer page needs a pass that reads all twelve in sequence.** |
| **9** | **⭐ "Cannabis" stays off the page — DECIDED 2026-08-09, not an oversight** | The page says *"licensed business and real-estate transactions"* and *"an industry where the rules differed by state."* The word itself appears nowhere. **A review pass flagged this against GATEKEEPER check 8** (*cannabis handled deliberately for the target, never left to the reader*) and **Jesse chose to keep it implicit.** That converts it from a gap into a decision, which is what check 8 actually demands — the check forbids *drift*, not discretion. **Residual, stated once and not re-litigated:** a reader works it out one click later at cannamls.com, so this buys a softer first impression rather than concealment. **Revisit when a real posting exists** — a federal or cleared target is the case where naming it up front, with the non-plant-touching distinction attached, becomes the stronger play. Do not quietly re-slant either way without asking. |
| 7 | **American spellings throughout** | British forms crept in repeatedly. Check `colour behaviour organis licence judgement favour recognis practis` — **plus `grey centre analyse defence`, added 2026-08-09** — before every deploy. **The checklist was itself the hole:** `grey` was not on it, and *"legally grey area"* sat in a reader-facing answer from rev 3d to rev 3l. A checklist that has never been audited against the file is a checklist that only catches what someone already thought of. |
| 10 | **⭐ Title Case for named deliverables in Selected work, sentence case for everything else** | **Added 2026-08-22** after Jesse flagged *"Seattle Vegan Group flier"* as reading wrong. Convention: a `.ttl` entry under Selected work is a proper name for a specific thing that exists — a film, an app, a document, a physical object — and gets full Title Case (*War of the Worlds: The True Story*, *Seattle Vegan Restaurant Guide*, **Seattle Vegan Group Flier**, **Blind Evaluation Protocol for LLM Instruction Sets** — the last two both corrected 2026-08-22, "flier" and "protocol...sets" were lowercase). **Exception: "This page."** Not a proper name, it's a self-effacing wayfinding label matching "You are here" in the same column — stays lowercase on purpose, do not "fix" it. Job/role titles under Experience and Education (Technical Lead, Venues; Co-Founder; Chief Information Officer and Partner) were already correctly Title Cased and are a separate category from Selected work — don't conflate the two when auditing. **Interview chip labels (the `k` field in `DATA`) are a THIRD, deliberately different convention** — sentence case / short-phrase style (*About me*, *A hard call*, *Why a job*) — never Title Case these, they are meant to read as conversational fragments, not titles. |

---

## 4. Revision log

| Rev | Date | What changed |
|---|---|---|
| **3o** | 2026-08-22 | **New Q&A: "Audience and distribution."** Answer covers the CannaMLS newsletter (built and ran it; ~11,000 subscribers after a list-hygiene cleanup, sourced to Jesse in-chat 2026-08-22 and logged in `Career_Master.md` — **do not state a higher number without asking him again**) and names the same flier→website→newsletter funnel now running at Seattle Vegan Group. **New featured Selected work entry: the Seattle Vegan Group flier**, with front and back images (`svg-flier-front.png` / `svg-flier-back.png`, copied from `../../Vegan Store/` and resized to 700px wide). **Corrected same session:** originally labeled "Designed," not "print-ready," because the flier's own handoff doc (`SVG_Flier_4x6_PRINT_HANDOFF.md`) flagged an unresolved font-swap gap before press. Jesse then said the run had already happened — **1,000+ copies printed and in circulation.** Label changed to **"In circulation,"** body copy updated to state the print run as fact. `SVG_Flier_4x6_PRINT_HANDOFF.md` itself was not touched this session and still shows the font gap as open as of 2026-07-23 — **that file is now stale and should be updated** (or at least checked) next time anyone opens it, since the print run evidently happened without that document being closed out. New `.flier` CSS block; images hidden in print via `@media print`. Footer date bumped to 22 August 2026.<br><br>**Question order revised twice this session, both times after Jesse pushed back on a first pass.** First pass placed "Audience and distribution" last, after "AI" — wrong, because it demoted AI from its documented deliberate-closer slot without earning that position. Three read-only sub-agent reviews (recruiter, UX/IA, editorial) were run against the full sequence; two independently recommended slotting the new entry right after **"Something I built"** instead, since it shares the same "what have you built" register and the same two organizations (CannaMLS, Seattle Vegan Group) — that placement was adopted. Jesse separately flagged that **"Under pressure," "Being corrected," and "A hard call"** ran three-in-a-row as CannaMLS-regulatory stories and read as repetitive, and proposed **"Under pressure" as the actual closer** instead of AI. Implemented: "Under pressure" moved to the final slot (displacing AI, which now sits at position 12); the duplicate phrase **"legally gray area"** — which appeared in both "Under pressure" and "A hard call" — was cut from "A hard call" (now "an industry the law hadn't caught up to yet") since "Under pressure" earns the line with its closing sentence. **⚠ This reverses the rev 3i/3j-era placement of "AI" as the fixed last chip.** That placement was never a hard rule (rules 1c and the ORDER comment govern AI's *content*, not its chip position) — it was an artifact of chronological appending that later revisions treated as settled without re-deriving it. **Chip label "Audience and distribution" renamed twice more, same session:** first to "11,000 subscribers" at Jesse's suggestion (a bare number, tested well in isolation), then a fresh sub-agent review round — recruiter, UX/IA, editorial, all re-reading the *revised* full sequence rather than the entry alone — converged that a bare number breaks the row's convention (every other label promises a topic/shape; a number promises nothing until clicked). Landed on **"Building an audience."** That same review round also surfaced a genuine split on the closer (recruiter + UX wanted "AI" back as the last chip; editorial defended "Under pressure" on the strength of its closing line) — **left as "Under pressure," Jesse's call, not resolved by the split.**

**"Being corrected" removed entirely, 2026-08-22, same session.** A dedicated research pass across `Career_Master.md`, `CAREER_REFERENCE.md`, both capstone `.txt` files, and the baseline resume/cover letter found **no non-CannaMLS story anywhere in the record** of a boss, client, regulator, or professor telling Jesse he was substantively wrong and how he responded — a genuine gap, not a search failure. Rather than keep a third CannaMLS-regulator story or force a weak substitute (the capstone IMAP outage is self-caught, not other-corrected; the falsified-prediction case study is self-correction via test data, not a person), Jesse chose to cut the entry outright. **Page is now 12 entries, not 13.** Two CannaMLS stories remain ("A hard call," "Under pressure"), no longer adjacent, different registers (policy decision vs. external crisis) — narrative repetition is resolved even though CannaMLS still appears in 8 of 12 source lines (mostly one-line context tags, not full stories).

**Flagged for a future session, not acted on:** the research pass found a stronger non-CannaMLS "Under pressure" candidate — a Seattle Videography client testimonial (David Allender) about Jesse keeping a publicist and interviewee calm through a live on-set disruption, `Career_Master.md` lines 1044–1052. Jesse said keep the best CannaMLS one for now; this is here if that changes. It also found a real "a hard call" candidate — the Cannabis Consultants wind-down after 7 months, `Career_Master.md` lines 1333–1346 — but the record is a one-line conclusion with no story behind it; would need Jesse to supply what actually happened (team pushback, cost) before it's usable.

Final order (12 entries): About me → What I want → What I'd bring → My weakness → Something I built → Building an audience → A mistake → A hard call → Why a job → Career change → AI → Under pressure. |
| **3p** | 2026-08-22 | **ATS keyword pass**, same session, following a sub-agent research review that cross-referenced the page against `Career_Master.md`/resume drafts for genuinely-earned-but-missing terms. Added, each traced to a real source rather than inserted for scanner coverage alone: **PCI DSS** (his own capstone scoped it out by name — "not subject to PCI DSS requirements... security can be ignored" — now in "Something I built"); **SEO** (his own statement, in-chat: "I did the SEO for everything I've ever made... takes pretty strong research back links, and I'm learning AI stuff" — now closes "Building an audience"); **Technical 2-Star Certified (Encore, badged via Credly)** — Jesse specifically asked for this on the Encore Experience bullet list, not the Certifications row, since it's tied to that job. **⚠ Not linked** — no public Credly badge URL is recorded anywhere in `Career_Master.md` (Credential ID column is blank, dated "— date needed —"), so it's plain text pending a real URL from Jesse. Two items from the same research pass were explicitly rejected: **FERPA** and **Google Analytics / Web Analytics** (real LinkedIn tags, zero backing story, would be pure keyword-stuffing). **On FERPA specifically:** Jesse confirmed real, direct experience — two specific student cases during the AmeriCorps Benefits Hub role — but both involve detail (family estrangement over gender identity; military-service dependency status) specific enough to risk identifying a real former student on a public page. Recommended adding just the bare term "FERPA" to the already-generic "What I want" answer instead of either story. **Jesse's call: skip FERPA and the stories entirely, not just the specifics.** Do not re-raise unless he brings it up.<br><br>**New Selected work addition: the *War of the Worlds* trailer**, which Jesse made himself — <https://www.youtube.com/watch?v=GAWWDDrrYsA>, verified live via Chrome 2026-08-22: 155K views, 804 likes, 14 years old, channel WAROFTHEWORLDSTRUE. Added as a **minimal branded thumbnail link**, not an embed — `wotw-trailer-thumb.png` (self-hosted, captured via a Chrome screenshot crop since the sandbox's network allowlist blocks `img.youtube.com` directly) with a CSS play-glyph in the site's own accent color, opens in a new tab like every other outbound link. **This is real engagement data on his *own* producer/editing work, distinct in kind from the Board Insiders attribution caution elsewhere in the record (that channel's current numbers belong mostly to a post-departure era; this trailer and its view count are entirely his). |
| **3q** | 2026-08-22 | **Trailer upgraded from link-out to play-in-place**, same session, after Jesse said linking out "wasn't ideal." Original 3p version was a static thumbnail wrapped in `<a href>`, navigating to YouTube. Now: a `<button>` with the same thumbnail; on click, JS swaps it for a `youtube-nocookie.com` iframe (`autoplay=1&rel=0`) and the video plays inline. **This is a real, flagged departure from the site's own "no tracking, no analytics, no cookies" design gate** — not a violation by accident, a deliberate trade Jesse asked for. The mitigation: nothing is fetched from YouTube until the button is clicked (zero third-party requests on page load, same as before), and `youtube-nocookie.com` is YouTube's privacy-enhanced embed domain — but once clicked, that is YouTube's player running with YouTube's own tracking, not the page's. README's design-decision table updated to state the exception explicitly rather than leave the blanket claim inaccurate. **No-JS fallback preserved** via `<noscript>`, a plain link to the YouTube watch page — so the "content doesn't need JavaScript" accessibility principle still holds; playing inline is progressive enhancement, not a requirement.<br><br>**Deploy confirmed live 2026-08-22.** `/contents/` API returned the same stale rev-3b snapshot the guide already documents as Trap 4 (`1b4aa9b3…`/28,744 bytes, README at 45 bytes) — ignored per the guide's own rule against trusting a single negative endpoint. Confirmed instead via the rendered page (`get_page_text`): all rev-3q tells present, footer dated 22 August 2026. Also **clicked the play button live** via Chrome — the youtube-nocookie.com iframe loaded and began buffering with no Error 153, confirming the `referrerPolicy` fix holds on the real https origin. |
| **3r** | 2026-08-22 | **Trailer embed params tightened for minimal chrome**, after Jesse asked whether the YouTube branding could be reduced further. Added `iv_load_policy=3` (no annotation cards), `fs=0` (no fullscreen button), `color=white` (progress bar, not YouTube red), `cc_load_policy=0` (captions off by default), `playsinline=1` (no forced native fullscreen on iOS). **`modestbranding` deliberately not added** — YouTube's embed terms require the logo stay visible on any embedded player, and the parameter has had no real effect on it since roughly 2018 regardless. Verified directly against Google's own player-parameters doc (fetched 2026-08-22, last updated by Google April 2026) rather than trusting Jesse's recollection or an AI-search summary that gave conflicting answers. Stated to Jesse as the honest ceiling: reduced, not eliminated. |
| **3s** | 2026-08-22 | **New Selected work image: a live screenshot of the Restaurant Guide**, framed in a minimal phone mockup (dark bezel, camera dot, home indicator — CSS only, no image asset for the frame itself) so it reads as an app screenshot rather than a browser window. `svg-guide-screenshot.png` (500×734) — captured via Chrome, cropped to exclude the Seattle Vegan Group site's own header/nav (first attempt included it; Jesse asked for just the app content "like it's on a phone," second capture cropped tighter to the widget box only). Whole phone frame is a link out to the live guide, matching the click-to-open pattern used elsewhere.<br><br>**"Career change" rewritten** — Jesse flagged that the "why move from media into IT" framing no longer held up once the page gained real, current content/audience/marketing material (Building an audience, the flier, the trailer). The old opening ("I didn't arrive at IT from a classroom. I arrived at the classroom from IT.") implied a clean departure from media that the rest of the page now contradicts. Question left as-is (real, researched hiring-manager question, rule 4 — don't invent a replacement). Answer reframed around "the two were never separate," folding the audience/content work into the same CannaMLS-era description instead of omitting it, and the closing line broadened from "security as baseline IT" to "security as baseline for any of it — content, marketplace or infrastructure." Nothing cut, only reframed; still cites the same two sources.<br><br>**Title-case pass** — see new hard rule 10 above for the convention. Fixed: "Seattle Vegan Group flier" → "Seattle Vegan Group Flier"; "Blind evaluation protocol for LLM instruction sets" → "Blind Evaluation Protocol for LLM Instruction Sets." **AI disclosure line added to the footer**, under "Last updated": *"This page is maintained with AI assistance — Claude, via Cowork, closely supervised. See the AI answer above for what that means in practice."* Points back to the existing "AI" answer rather than duplicating its content — footer states the fact, the AI chip carries the explanation. |
| **3n** | 2026-08-09 | **All twelve answers rewritten in the plain, contracted voice the AI answer already used.** Jesse: *"lets make it more human readable and not do that, metaphore is also okay."* The register split ran the wrong way — eleven formal answers and one human one — and the human one was the good one. **The file got shorter.** Structural fixes from the full review pass: **both near-verbatim duplications removed** ("three things running" now only in *Why a job*; no-account posting only in *A hard call*, where it is load-bearing); **"My weakness" now leads with the real weakness** — framework vocabulary — with the consensus-building habit demoted to second and no longer resolved two sentences after it is raised, and the unprompted *"what I would not offer as a weakness is follow-through"* brag cut; **"Why a job" stopped undercutting itself** (*"and after that, who knows"* gone; *"I like having a few things going at once"* reframed to answer the stability objection rather than feed it); **"Being corrected" expanded** — it was the shortest answer on the page and the only story validated by an outside authority, so it gained the counterfactual paragraph; **four consecutive counted openers** ("Three things" / "Three things" / "Two things" / "Biggest first") broken up. Two metaphors added, both load-bearing: *a deeper keel*, and *low barrier at the door, high bar at the gate*. |
| **3m** | 2026-08-09 | **IMDb moved from the Options bar into the header contact line**, after LinkedIn and GitHub. Jesse: *"the imdb link should be the last next to the linkedin, and github, they are profiles. its not a function."* **The rev 3l placement was wrong and the guide told me to put it there** — see backlog 11. The bespoke `<a class="opt">` and its CSS override are gone; it is now a plain link like the other two. **"Under pressure" answer rewritten** to carry the substance from `Career_Master.md` §11 — not plant-touching, interstate facilitation as the actual exposure, Cole priorities / Section 230 / free speech as an assembled framework, Cole still the working standard after rescission, and cooperating with regulators as a deliberate choice. **Rev 3l had corrected the duration and stopped there, which was the error Jesse caught.** `llm-instruction-evals` entry rewritten again in plain language and now names the self-improving loop: friction → candidate rule → control-tested → kept or dropped. **⚠ Stale claim removed from the AI answer** — it said the write-up was published but *"the current version is private and mostly notes to myself,"* which stopped being true when the repo went to three phases. Now: the **method** is published, the **rule set** stays private. **Four more clumsy sentences fixed**, three of them the same construction Jesse flagged twice: *"I would rather be the person who reports his own outage than the person who turns out to have had one"* → *"I would rather report my own outage than have someone else find it later"*; *"I would rather say that plainly than have you find it"* → *"Better you hear it from me"*; *"the ability to be ill for two weeks"* → *"the ability to take two weeks off sick"*; and one of my own from earlier this session. **Full twelve-answer read completed** at Jesse's push — *"sounds like you didnt do a second pass."* He was right; there had been no second pass. It found the **"I would rather X than Y" tic, seven instances, two consecutive answers opening with the same construction** (new hard rule 8), and five more edits: both *"Three things, and I would rather…"* openers, *"rather tell you what I have not done than let you discover it later,"* *"rather name both than have you find them,"* and *"anything I have published that nobody has adopted"* → *"but nobody has picked up."* |
| **3l** | 2026-08-09 | **IMDb added to the Options bar** (backlog 11, unblocked since 08-07) as an `<a class="opt">` in last position — it picks up `target="_blank"`, `rel`, and the `↗` marker from the existing externalize pass and the `a[target="_blank"]::after` rule; only a `text-decoration:none` override was needed. **Two phrasings fixed at Jesse's flag:** *"I am not precious about being the one who spots the problem"* → *"It does not matter to me who spots the problem"*; *"the simplest way to know I was talking to who I thought I was talking to"* → *"the simplest way to confirm they were who they said they were."* **⚠ Two factual corrections found while editing, both live since rev 3d** — the regulatory episode said *"a few weeks"* against an actual **~six months**, and **five British spellings survived rule 7**, including a reader-facing *"legally grey area"* in the "A hard call" answer. Rule 7's own checklist did not include `grey`; it does now. `llm-instruction-evals` entry rewritten for the repo's phase-3 update. Footer date bumped. |
| **1** | 2026-08-07 | First build. Card layout, blue accent, nine invented questions. Film credits misattributed to Seattle Videography (they were Pendragon Pictures). |
| **2** | 2026-08-07 | Editorial redesign: credit-block grid, hairline rules, accent removed entirely. Film credits corrected. Archived. |
| **3** | 2026-08-07 | Affordance rebuild after *"the top buttons don't feel like buttons."* Five stacked cues. Warm palette trialled. Print stylesheet, favicon, OG tags, pre-loaded first answer, `aria-pressed`. |
| **3b** | 2026-08-07 | Clean white, dark pea green accent `#3C4F1B`. Long-form answers — the "imaginary limit" fix. Two accuracy corrections (§6). **SHA-verified live.** |
| **3c** | 2026-08-07 | Evidence re-weighted per Check 0. Restaurant guide promoted from absent to prominent; capstone cut to one clause; `In use` above `Published`. |
| **3k** | 2026-08-07 | **Links inside answers + new-tab behaviour.** 17 links across the twelve answers, **first mention only** per answer. Rendered through a `[label](url)` parser that escapes first and accepts only `http`/`https`. All external links now `target="_blank" rel="noopener noreferrer"` with a `↗` marker; `mailto:` excluded. |
| **3j** | 2026-08-07 | **AI answer rewritten in plain language.** Jesse: the previous draft was *"a foreign language"* only an AI engineer would follow. Now leads on the two things any reader grasps — an app people use, and this page — with the methodology demoted to one jargon-free paragraph. "Built with AI" labels added to the guide and This page entries in Selected work. |
| **3i** | 2026-08-07 | **Terminal theme + random question.** Theme button became a three-way cycle (dark → light → terminal), always advertising the next one. Terminal is opt-in phosphor monospace. **Print now resets all themes to ink-on-paper**, which also fixed dark mode leaking grey text into print. Random-question button never repeats the current answer and moves focus to it. |
| **3h** | 2026-08-07 | **Options bar** — labelled utility row above the name: theme, Print / PDF, Copy email. **Fixed a real print bug**: printing captured only the single open answer, so a printed copy was missing eleven of twelve. A hidden `#printall` block now holds every Q&A and is revealed only in print. Footer gained a **Last updated** line. |
| **3g** | 2026-08-07 | **Theme toggle, dark by default.** Switched from `prefers-color-scheme` to a `data-theme` attribute on `<html>` so the default is a choice rather than the reader's OS setting. Inline `<head>` script applies the stored preference before first paint (no white flash). Choice persists in `localStorage`, `try/catch` wrapped. Chip shadows became variables — the hardcoded black ones were invisible on dark. |
| **3f** | 2026-08-07 | **Build-neutral sweep of the metadata.** Title, OG/Twitter tags, JSON-LD description and `og.png` no longer frame him as *"IT, compliance and operations"* — they describe the record instead. `knowsAbout` broadened across media, product and community work. **This closes the F6 / A1 tension that had been open since the first draft.** |
| **3e** | 2026-08-07 | **Social and link metadata.** Full Open Graph set + `og.png` (1200×630), Twitter `summary_large_image`, canonical URL, `theme-color`, `profile:` tags, and JSON-LD `Person` schema with `sameAs`, `worksFor`, `alumniOf`, `knowsAbout`. Shared links now render a card instead of bare text. |
| **3d** | 2026-08-07 | **Reframed as an interview.** Questions replaced with researched ones, 12 total. Declines and disclaimer retired to phase 2. Jesse's per-question notes folded in. *War of the Worlds* added as first Selected work. README written. This folder created. |

---

## 5. Phase 1 vs phase 2

**Phase 1 (now):** a written interview. Twelve questions, hand-written answers, no model call.
**Phase 2 (maybe):** a live-model version behind a Cloudflare Worker holding the API key server-side.
Jesse already administers that account.

The `ok` flag and its `Not in record` rendering remain wired in `index.html`, **dormant**, so phase 2
does not have to rebuild the mechanic.

### Preserved for phase 2 — the retired decline answers

> **Q: What is your Kubernetes experience?** *(ok: 0)*
>
> "There is nothing in my record about Kubernetes, so I am not going to answer as though there is.
>
> My infrastructure experience is edge security and DNS on Cloudflare, hosting and email
> administration, and networked audio over IP. Container orchestration is not part of it. If it is
> central to the role, that is a real gap and you should weigh it as one."

> **Q: Have you managed a departmental budget?** *(ok: 0)*
>
> "My record does not contain budget figures I can stand behind, so I am not going to estimate one.
>
> What I can say is narrower. I have run cost-constrained projects, including selecting a zero-cost
> security solution for a non-profit after comparing paid alternatives and explicitly weighing
> vendor lock-in. That is procurement judgment. It is not the same thing as owning a departmental
> budget, and I would rather draw that line than blur it."

### Retired disclaimer copy

> "Answers are written in advance and shown verbatim. There is no model call and nothing is
> generated, so this page cannot invent a credential. Questions outside the record are declined."
>
> "This page answers from a fixed record. It does not generate text and cannot invent a credential."

**In phase 2 all of this becomes load-bearing rather than decorative.** Any employer who has used an
AI resume for thirty seconds will try to break it, and a system that always has an answer is a
system that is making things up. **The declines are what make every other answer credible.**

**If phase 2 is built:** run `llm-instruction-evals` against it and publish the scorecard beside it.
That would be more persuasive than the resume itself — it is the gap between claiming AI experience
and demonstrating the constraints were measured.

### Also rejected, with reasoning, so it does not get rebuilt

**A free-text ask box.** Higher raw legibility than buttons, since a search input is the most
universally recognised interactive element on the web. **Rejected because it sets an LLM
expectation**, and to a non-technical reader a principled refusal is indistinguishable from a broken
chatbot. Buttons never promise more than a fixed-answer design can deliver.

---

## 6. Errors that reached the live page

Recorded because they were public, not to flagellate. Both are logged as Candidate #43 in
`../../AI settings/SETTINGS_LOG.md`.

**1. The Oklahoma attribution.** The page said *"I identified a gap in our listing disclosures."*
**He did not — the regulator called and raised it.** The source quote never said who noticed;
`Career_Master.md` rendered the episode as a five-stage lifecycle table, stage one needed an actor,
and Jesse's name got pulled into the empty cell. Now states plainly: *"He found it, not me."*
**The corrected version is the stronger claim**, because it survives being checked.

**2. The `.gov` motive.** The page attributed a threat model to him — *"impersonating an official is
a normal way to extract business records without a warrant."* **His actual reasoning was practical:**
*"They would call and I needed to verify who they were, so that seemed like the simplest way."* Now
describes what he did, then names the concept separately as an observation.

**3. Film credits under the wrong company** (rev 1). *War of the Worlds* and *10 Days in a Madhouse*
were **Pendragon Pictures**, not Seattle Videography. Assembled from a thematic section of the master
rather than from the employment history. **Build role entries from §2, not from topic sections.**

---

## 7. Backlog

1. **PDF export** — **substantially addressed in rev 3h, not fully closed.** The Print / PDF
   button plus the print stylesheet lets a reader produce a complete PDF via the OS print dialog
   ("Save as PDF" on both macOS and Windows), and the printed copy now contains **all twelve**
   answers. **What is still missing is a hosted `.pdf` file to attach to an application** — some
   employers require a file upload and some ATS cannot ingest a URL at all. Deliberately not built
   as a second document, because a separate PDF drifts out of sync with the page; if one is ever
   needed, generate it *from* this page rather than authoring it separately.
2. ~~OG preview image~~ ✅ rev 3e — `og.png`, 1200×630
3. ~~JSON-LD `Person` schema~~ ✅ rev 3e. **`hasCredential` deliberately omitted** until the degree
   confers; add the B.S. and the 22 certifications then
4. **The conflict question** — *"tell me about a disagreement with a coworker"* is commonly asked and
   there is **no material in the record for it.** Needs a real story from Jesse or it stays off
5. **Section heading `Ask` → `Interview`** — would tell a reader what they are looking at faster
6. **Serif vs sans for the name** — serif drafted, not adopted
7. **Test the button affordance on a real person** — the 8-in-10 target is asserted, never measured
8. ~~Header tagline vs build-neutral A1~~ ✅ rev 3f. The page header carries no tagline at all
   (name and contact only), and the metadata now describes the record rather than a target role.
   **F6 closed.**
9. **Phase 2** — live model via Cloudflare Worker, with the eval scorecard beside it
10. ~~Verify rev 3k deployed~~ ✅ **2026-08-09 — confirmed at the content level, not by hash.** The
    live page renders the Options bar, twelve linked answers, the "Built with AI" labels and the
    full OG set, all of which exist only in 3j/3k. **The SHA check remains unavailable** — see
    Trap 4. If a true hash match is ever needed it requires an authenticated API call or a clone.
11. ~~**🎬 Add IMDb to the Options bar, as the LAST item**~~ ✅ **DONE — but NOT in the Options bar.**
    Built there in rev 3l and **moved to the header contact line in rev 3m**, after LinkedIn and
    GitHub.

    > **⚠ This entry was wrong, and it was wrong in a way worth keeping.** It recorded a
    > *placement* — "the Options bar, as the LAST item" — and I implemented the placement without
    > re-deriving whether it was right. Jesse's correction: **the Options bar holds functions
    > (theme, print, copy, random). IMDb is a profile, and profiles already have a home** next to
    > LinkedIn and GitHub. The reasoning below for *why IMDb earns a permanent slot* was always
    > sound; only the location was wrong.
    >
    > **The general lesson:** a backlog entry is a decision made in an earlier session with less
    > context. Its *justification* is durable; its *implementation detail* is a guess that should
    > be re-checked against the page as it now stands. This one had sat unbuilt for two days while
    > the Options bar filled up with controls, which is exactly what made the placement stale.

    Original entry retained for the reasoning:

    ```
    https://www.imdb.com/name/nm3561148/
    ```

    Confirmed 2026-08-07 via browser: page title returns **"Jesse Zesbaugh - IMDb"**. *(IMDb blocks
    plain fetches — it needs the Chrome tools, and `get_page_text` returns nothing because the page
    is JS-rendered. The tab title was sufficient to confirm identity.)*

    **Build notes:** fourth `.opt` button, after Random question. It is a plain outbound link, not
    a control, so consider `<a class="opt">` rather than `<button>` — and it will pick up
    `target="_blank"` and the `↗` marker automatically from the externalize pass.

    **Why it earns a permanent slot:** it is the only credential on the page **verified by a third
    party rather than asserted by him**. Everything else a reader has to take on trust or check via
    a link he chose. That is a real difference and it belongs somewhere persistent rather than
    buried in Selected work.
12. **Keep the footer `Last updated` current** — now step 2 of the §2 deploy procedure. Flagged
    because it was already at risk of drifting: a hand-maintained date on a live page is worse
    than no date once it goes stale.
13. **Dark `og.png`** — the page now defaults to dark but the social card is white, so the preview
    and the page look like different sites. A dark card would also stand out more in the mostly
    white feeds of LinkedIn and Slack. **Open question for Jesse; one card serves everyone, so it
    is a straight choice.**

---

## 8. Time-sensitive — content that will become wrong

Full table in `../CAREER_REFERENCE.md`. The two that touch this page directly:

- **Co-op incorporation (imminent).** *"in formation"* and *"nothing is filed yet"* appear in the
  **"Why a job"** answer and the Seattle Vegan Group experience entry. Both go false on filing.
- **PenTest+ / degree (Oct 2026).** *"expected"* appears in Education and *"I finish in October"* in
  the **"About me"** answer.

---

## 9. Current state

**Rev 3o — edited locally 2026-08-22, in progress. ⏳ AWAITING JESSE'S REVIEW, THEN UPLOAD.**
`README.md` and `og.png` unchanged since rev 3k. **This upload needs FOUR files**, not one:
`index.html`, `svg-flier-front.png`, `svg-flier-back.png` — confirm the two PNGs actually landed in
`Resume_Site/` before uploading (copied and resized this session; see §1) — and verify whether rev
3n ever went up (see below), since 3n's content ships as part of 3o either way.

**Superseded: rev 3n** — edited locally 2026-08-09, approved by Jesse, upload status not confirmed
before this session started. Local `index.html` at 3n was **55,878 bytes**. Its content is carried
forward into 3o.

**Tells on the live page**, in the order they are quickest to check:

1. **Twelve chips, not thirteen** — "Being corrected" is gone
2. **The last chip reads "Under pressure,"** not "AI" — AI sits second-to-last
3. **"Building an audience"** is the sixth chip, right after "Something I built" — 3o-only
4. **The flier images** appear under Selected work, labeled "Designed" — 3o-only
5. **IMDb in the header line** beside LinkedIn and GitHub — *not* in the Options bar
6. **"About me" opens** *"I've spent twenty years…"* with a contraction — the whole page is in that
   voice now, so the first rendered answer is the tell
7. **"My weakness" opens** *"Vocabulary, and it's the one that will cost me first."*
8. Footer reads **22 August 2026**

**Superseded: rev 3m** (56,248 bytes) — never uploaded. Its content is carried forward in 3n.

**Rev 3l — uploaded by Jesse 2026-08-09, confirmed live at content level, now superseded.**
Local `index.html` was **55,950 bytes**. `README.md` and `og.png` were unchanged from rev 3k and were
not re-uploaded.

**How it was confirmed:** the rendered page carries all three rev 3l tells — **IMDb in the Options
bar**, the rewritten `llm-instruction-evals` entry naming the control arm, and the footer reading
**9 August 2026**. **Not hash-confirmed**; the blob SHA was never computed locally, so §2 step 2 was
skipped this round.

> **🪪 Trap 4, observed again in the same session and worth the note.** Immediately after the upload,
> `https://jzesbaugh.github.io/resume/` — the **directory** URL — still served rev 3k, while
> `https://jzesbaugh.github.io/resume/index.html` served rev 3l. **The two paths cache separately.**
> Fetching the explicit `index.html` path is the faster way to see a fresh deploy; the directory URL
> catches up on its own. `Cmd+Shift+R` for a browser.

**Rev 3k — deployed, confirmed at content level 2026-08-09.** The recorded expectations were
`index.html` `76b82dfa…` / 55,128 · `README.md` `1eaf12f8…` / 9,073 · `og.png` `422af696…` / 44,030.
**These were never hash-confirmed** — `/contents/` returned a stale payload naming rev 3b, and the
other endpoints stayed empty. The live render carries 3j/3k-only features, which is why the
revision is treated as deployed. **Content-level, not hash-level. Do not upgrade that claim.**

**Last hash-verified deployment remains rev 3d** — `index.html` `44edf020…` / 36,571 and `README.md`
`4c191eb0…` / 5,270.

*Method note: rev 3d was the first deployment in this project confirmed by hash rather than by
reading a page. The two attempts before it were called wrong in **opposite directions** from
cached fetches. Use §2 every time, and when the API is unavailable, say so rather than guess.*
