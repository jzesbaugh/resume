# jzesbaugh.github.io/resume

An interactive resume. Twelve interview questions, answered in advance, in one HTML file.

**Live:** <https://jzesbaugh.github.io/resume/>

```
index.html    the whole thing — markup, styles, answers
og.png        1200×630 social preview card
README.md     this file
```

---

## What this is

A resume you read by asking it things.

The questions are the ones hiring managers actually ask — *tell me about yourself*, *what's
your greatest weakness*, *tell me about a mistake you made* — and the answers are written the
way I would say them out loud, at the length a real answer takes.

It is not a chatbot. Every answer is written in advance and shown verbatim. That is a
deliberate limitation rather than a missing feature; see [Honest limits](#honest-limits).

## Why build it this way

A one-page PDF forces everything into fragments. *"Regulatory liaison across four states; all
inquiries closed with no violation found"* is true, and it is not remotely what I would say if
you asked me about it. It leaves out that they phoned rather than wrote, that I could not
verify who was on the line, and what I did about that — which is the entire point of the story.

This format has no length constraint, so the answers get to be answers.

The trade is that you have to guess the questions. So I looked up which ones actually get
asked rather than inventing a set that flattered the material I happened to have lying around.

## Design decisions

| Decision | Why |
|---|---|
| **One file, no dependencies** | No build step, no npm, no CDN, nothing to rot. Download it and it works offline. |
| **Content doesn't need JavaScript** | Everything except the question buttons is plain HTML. Screen readers, text browsers, crawlers and locked-down corporate machines all get the full resume. |
| **No tracking, no analytics, no cookies** | Nobody needs to know you looked. |
| **No model call, no API key** | Nothing is generated at runtime, so nothing can be invented. |
| **It prints** | Somebody is going to print this. There's a print stylesheet, and the interactive bits drop out cleanly. |
| **Email only** | It's a public page. Phone and street address are deliberately absent. |
| **Every answer cites its source** | So a claim can be checked rather than taken on faith. |
| **One accent color** | Dark pea green. Enough to have a point of view, restrained enough for a government hiring panel. |
| **It describes the record, not a target role** | The title says what I have done, not what I am applying for. I have not picked a lane, and a page that picks one for me narrows the conversation before it has started. |
| **Real link metadata** | Full Open Graph set, a 1200×630 card, Twitter `summary_large_image`, canonical URL, and JSON-LD `Person` schema. A link that unfurls as bare text looks like nobody finished it. |

## Accessibility

- Real `<button>` elements with `aria-pressed` and `aria-controls`, not clickable `<div>`s
- Visible focus rings, keyboard navigable
- Respects `prefers-color-scheme`, including a dark theme
- Body text 15–16px, line-height 1.65+
- Contrast checked against the background in both themes

The buttons went through a rewrite specifically because an earlier version styled them as
underlined text and nobody could tell they were interactive. Five stacked cues now: an explicit
instruction line, filled backgrounds rather than outlines, a chevron, a hover lift, and the
first question pre-selected so the cause and effect is visible before you click anything.

## Make it your own

MIT licensed. Genuinely, take it.

1. Fork this repo, or just download `index.html`
2. Edit the `DATA` array near the bottom. Each entry looks like this:

```js
{k:"Chip label", ok:1,
 q:"The question, phrased the way a person would ask it.",
 a:"Your answer.\n\nBlank lines become paragraphs.",
 s:"Where this comes from"}
```

3. Replace the experience, education and selected-work sections — they're plain HTML, no
   templating
4. Change `--accent` in `:root` if green isn't your color. It's used in exactly three places
5. **Replace `og.png`** and update the `og:` / `twitter:` URLs in `<head>` to point at your
   domain. They must be **absolute** URLs — relative paths don't resolve for social scrapers, and
   a missing image fails silently
6. Update the JSON-LD `Person` block, or delete it
7. Commit as `index.html` at the repo root, enable GitHub Pages, done

After the first deploy, run the URL through LinkedIn's Post Inspector or Meta's Sharing Debugger.
They cache preview data per URL for a long time, including caching the *absence* of an image, so
a link shared before the card existed will keep showing the old version until you force a
re-scrape.

Two things worth keeping if you fork it: **write the questions first, from what people actually
ask**, not from what you want to say. And **put your real limits in.** A page that only makes
you look good reads like a page that only makes you look good.

## Honest limits

- **It can't answer anything I didn't anticipate.** Twelve questions is not an interview. It's
  a head start on one.
- **It can't answer "why do you want to work *here*".** That's the best question in any
  interview and a generic page structurally cannot do it. Ask me directly.
- **The `ok:0` decline mechanic is dormant.** It renders a "Not in record" tag instead of "From
  record", and nothing currently uses it. It was built for a possible later version with a live
  model behind it, where refusing out-of-record questions would be a real guarantee rather than
  decoration. On a hand-written page there's nothing to refuse, so the decline answers came out.
- **No PDF yet.** Some employers won't accept a URL, and some applicant tracking systems can't
  ingest one at all. Known gap, and currently the only significant one.
- **The schema has no `hasCredential`.** That's the natural place to list the degree and the
  certifications, and it stays empty until the degree actually confers in October. Structured
  data asserting a credential you don't hold yet is worse than omitting it.

## Colophon

Written August 2026. Built and revised in a single long session, including a few rounds where
the first attempt was wrong and had to be reversed — the question set was originally invented
rather than researched, and the weighting put a university capstone above a national platform
because there happened to be more documentation about the capstone. Both got fixed. That felt
worth mentioning on a page about being straight with people.

## License

MIT. The scaffolding is yours. The answers are mine.
