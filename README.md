# Trip Itinerary Page

A [Claude Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) that turns a day-by-day trip plan into a single-file, shareable itinerary website — no build step, no backend, works on a phone.

Built out of a real trip (see: three weeks of back-and-forth planning a friend's first visit to Tokyo, argued about every train line and diet-of-udon decision along the way). Generalized here so it works for any trip, any traveler, any destination.

<img width="800" height="450" alt="ScreenRecording2026-09-06at11 19 11AM-ezgif com-video-to-gif-converter" src="https://github.com/user-attachments/assets/c3a0ad6d-6f12-46d7-a01b-ca8bbfb05bce" />


*Above: a fictional week in Lisbon, built with this skill.* **[▶ Open it live](https://adedeepfishing.github.io/trip-itinerary-page/examples/demo-lisbon-trip.html)**

## What it makes

- **Sticky day-by-day tabs** that auto-scroll to stay in sync as you scroll the page
- **A live countdown** — days until arrival, then "Day N of the trip" once it starts, then a wrap-up message after
- **Timeline-style activity cards** per day, with tags for free / needs booking / price / transit / solo-friendly / a custom "highlight" tag for whatever the trip's own flavor is
- **A booking/prep checklist**
- **An "essentials" panel** — meeting points, emergency numbers, weather plan, cash notes — with a built-in pattern for keeping a real home address out of a page that might get shared or hosted publicly
- **Two themes**, swappable with one HTML attribute: `kawaii-pink` and `neutral-modern`

## Demo

[`examples/demo-lisbon-trip.html`](examples/demo-lisbon-trip.html) is a fully fictional trip — no real people, places, or dates — built to show every feature at once: a full 7-night week, a custom destination-flavored theme (azulejo blue + terracotta, built on top of the two base themes as a worked example of "add a third theme"), all six tag types, the checklist, and the essentials panel using the privacy pattern for real. [See it live](https://adedeepfishing.github.io/trip-itinerary-page/examples/demo-lisbon-trip.html), or open the file directly in a browser.

## Installing it

**Claude Code** — clone straight into your skills folder, and the directory name becomes the skill name:

```bash
git clone https://github.com/AdeDeepFishing/trip-itinerary-page.git ~/.claude/skills/trip-itinerary-page
```

That makes it available in every project. To scope it to one project instead, clone into `.claude/skills/` inside that repo.

**claude.ai (web / desktop)** — download `trip-itinerary-page.skill` from [Releases](https://github.com/AdeDeepFishing/trip-itinerary-page/releases/latest), then upload it under Settings → Capabilities → Skills.

**API / Agent SDK** — point your skills directory at a checkout of this repo, or bundle `SKILL.md` and `assets/` with your agent.

## Using it

**With Claude:** just describe the trip — dates, who's going, what you've already planned — and ask for a page. Claude will walk through `SKILL.md`, start from `assets/template.html`, and fill it in. It'll also use this skill to update the page later when your plans inevitably change.

**By hand:** copy `assets/template.html`, read the comments inside it, and fill in the bracketed placeholders yourself. No dependencies — open it in a browser to preview.

## Deploying the page you make

It's one HTML file with no build step, so any static host works. Two easy paths:

**Vercel** — good when you want a link in under a minute, and a URL that's not tied to a public repo.

```bash
npm i -g vercel
vercel        # preview URL
vercel --prod # production URL
```

Run it from a folder containing your page renamed to `index.html`. Or skip the CLI entirely and drag the folder onto [vercel.com/new](https://vercel.com/new).

**GitHub Pages** — good when the page already lives in a repo you're pushing to anyway. In the repo: Settings → Pages → Source: *Deploy from a branch* → `main` / root. The page lands at `https://<user>.github.io/<repo>/<path>.html` a minute or so later. (That's exactly how the demo above is hosted.)

Either way, if it's Git-connected, **pushing is what triggers a redeploy** — editing the file locally alone won't update the live page.

One thing to decide before you host anywhere: a deployed URL is guessable and unlisted ≠ private. Run the privacy pass in `SKILL.md` §2 first, especially if anyone's home address is in the page.

## Structure

```
trip-itinerary-page/
├── SKILL.md              — instructions for Claude
├── assets/
│   └── template.html     — the starting point, fully commented
├── examples/
│   └── demo-lisbon-trip.html — a fictional filled-in example
├── docs/
│   └── demo.gif          — the recording at the top of this README
├── LICENSE               — MIT
└── README.md             — this file
```

## Notes from building it

- **Timezones and date rollovers are the #1 bug.** A flight that departs at 11am can land the *next* calendar day at the destination. Confirm actual arrival date + destination timezone before hardcoding the countdown — getting this wrong means someone ends up waiting at an arrivals gate for a flight that isn't even airborne yet. (Yes, this happened. Twice.)
- The nav bar auto-scroll fix matters more than it looks — without it, the active-day tab quietly scrolls off the edge of the tab strip on any trip longer than about five days.
- If the page might go public, sanitize addresses *everywhere* they appear, not just in the one card obviously meant for it — a redacted "Home base" line next to three casual real-name mentions elsewhere defeats the purpose.

## License

MIT — take it, fork it, theme it however you want. See [LICENSE](LICENSE).

---

made with ♥ by [Yanwen](https://www.yanwensworld.com/)
