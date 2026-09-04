# jzesbaugh.github.io/resume

An interactive resume. Thirteen interview questions, answered in advance, in one HTML file.

**Live:** <https://jzesbaugh.github.io/resume/>

```
index.html                 the whole thing — markup, styles, answers
og.png                     1200×630 social preview card
svg-flier-front.png        Selected work image — a printed piece, front
svg-flier-back.png         Selected work image — a printed piece, back
svg-guide-screenshot.png   Selected work image — phone-mockup screenshot of a live app
wotw-trailer-thumb.png     Selected work image — self-hosted trailer thumbnail, plays inline
README.md                  this file
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
| **No tracking, no analytics, no cookies — by default** | Nobody needs to know you looked. One exception: the *War of the Worlds* trailer under Selected work is a plain local thumbnail until you click it. Clicking swaps it for a `youtube-nocookie.com` embed and plays the video right there. Nothing is fetched from YouTube unless you choose to play it — but once you do, that's YouTube's player, not mine. |
| **No model call, no API key** | Nothing is generated at runtime, so nothing can be invented. |
| **It prints, and it prints everything** | Somebody is going to print this. The stylesheet resets whichever theme you're in back to ink on paper, drops the buttons, and outputs **every** answer — not just the one that happened to be on screen. Your browser's print dialog will Save as PDF. |
| **Three themes, light by default** | Light, dark, and a phosphor terminal theme. Your choice persists. Light as the default deliberately overrides your system setting, which is only defensible because the toggle is right there. |
| **Email only** | It's a public page. Phone and street address are deliberately absent. |
| **Every answer cites its source, and links it** | So a claim can be checked rather than taken on faith. Two kinds of link, on purpose: an organization mentioned by name links to that organization's own page, every time it's named; a specific project or client mentioned inside a story links to *my own* write-up of that work, never to the client's site — a client link sends a reader away from evidence and toward a page with none of it. |
| **External links open in a new tab** | A deliberate departure from the usual advice, which is that you shouldn't hijack someone's navigation — and I'd normally agree. A resume has a specific failure mode though: a reader clicks through to Wikipedia and never comes back. Keeping the page open is worth more here than the principle costs. The condition that makes it acceptable is the small ↗ marker on every outbound link, so you're told before you click rather than after. `rel="noopener"` on all of them. `mailto:` is excluded, since it would leave a blank tab behind. |
| **One accent color** | Dark pea green. Enough to have a point of view, restrained enough for a government hiring panel. |
| **It describes the record, not a target role** | The title says what I have done, not what I am applying for. I have not picked a lane, and a page that picks one for me narrows the conversation before it has started. |
| **Real link metadata** | Full Open Graph set, a 1200×630 card, Twitter `summary_large_image`, canonical URL, and JSON-LD `Person` schema. A link that unfurls as bare text looks like nobody finished it. |

## Options bar

Four controls, sitting above the name:

- **Theme** — cycles dark → light → terminal, always labelled with the *next* one so a click has a
  predictable result
- **Print / PDF** — opens the print dialog, where both macOS and Windows offer *Save as PDF*
- **Copy email** — confirms in the button itself rather than throwing an alert
- **Random question** — never repeats the one already showing, and moves focus to the answer

Deliberately a plain labelled row rather than a dropdown. Four controls don't justify the focus
trapping, escape handling and click-outside logic a real menu needs, and a row is more accessible.

## Accessibility

- Real `<button>` elements with `aria-pressed` and `aria-controls`, not clickable `<div>`s
- Visible focus rings, keyboard navigable
- Random question moves focus to the answer panel, so a screen reader announces what changed
  instead of leaving you sitting on the button
- Body text 15–16px, line-height 1.65+
- Contrast checked against the background in all three themes. The terminal green is deliberately
  **not** the classic `#33FF33` — that vibrates against black and is unreadable at paragraph
  length. Body text there sits around 14:1

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
4. Change `--accent` in `:root` if green isn't your color. Each theme is a `:root[data-theme="…"]`
   block of variables — edit those, or delete the ones you don't want and simplify the cycle array
   in the theme script
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

- **It can't answer anything I didn't anticipate.** Thirteen questions is not an interview. It's
  a head start on one.
- **It can't answer "why do you want to work *here*".** That's the best question in any
  interview and a generic page structurally cannot do it. Ask me directly.
- **The `ok:0` decline mechanic is dormant.** It renders a "Not in record" tag instead of "From
  record", and nothing currently uses it. It was built for a possible later version with a live
  model behind it, where refusing out-of-record questions would be a real guarantee rather than
  decoration. On a hand-written page there's nothing to refuse, so the decline answers came out.
- **No hosted PDF file.** Print / PDF will give you a complete one through your browser, but there
  isn't a `.pdf` sitting in the repo for employers whose upload form demands a file. That's a
  deliberate omission rather than an unfinished one — a separate PDF drifts out of sync with the
  page, and then there are two resumes that disagree.
- **The schema has no `hasCredential`.** That's the natural place to list the degree and the
  certifications, and it stays empty until the degree actually confers in October. Structured
  data asserting a credential you don't hold yet is worse than omitting it.

## Colophon

Written August 2026. Built and revised in a single long session, including several rounds where
the first attempt was wrong and had to be reversed. The question set was originally invented
rather than researched, and missed the most-asked interview question entirely. The weighting put
a university capstone above a national platform because there happened to be more documentation
about the capstone. And printing quietly produced a resume missing all but one of its answers
for longer than I'd like to admit.

All fixed. It felt worth mentioning on a page about being straight with people.

## License

MIT. The scaffolding is yours. The answers are mine.
