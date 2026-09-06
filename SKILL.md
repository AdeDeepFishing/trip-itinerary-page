---
name: trip-itinerary-page
description: Build a single-file HTML website that turns a multi-day trip plan into an interactive, shareable itinerary page — sticky day-by-day navigation that auto-scrolls to the active day, a live countdown to arrival/departure, a booking/prep checklist, and a quick-reference "essentials" panel (meeting points, emergency numbers, local tips). Use this skill whenever the user is planning a trip — for themselves, a visiting friend, family, or a group — and wants to organize it as something they or someone else can check on their phone throughout the trip. Trigger this even if the user just asks for "a page," "a site," or "something I can send them" for a trip that's already been discussed in the conversation — a multi-day itinerary is the signal, the word "itinerary" doesn't need to appear. Also use this to UPDATE an itinerary page already built earlier in the conversation when plans change (flight delays, weather, swapped days, cancelled bookings).
---

# Trip Itinerary Page

Turns a day-by-day trip plan into a single HTML file: sticky tabs per day, a countdown banner, timeline-style activity cards with tags (free / needs booking / price / transit / solo-friendly / highlight), a prep checklist, and an essentials panel. No build step, no dependencies — one file, works from a phone, easy to redeploy anywhere (Vercel, Netlify, GitHub Pages, or just opened locally).

Start from `assets/template.html`. Do not build this from scratch — the template already has tested JS for the two trickiest parts (the nav auto-scroll and the countdown/date logic below), and reinventing them tends to reintroduce bugs that are already fixed here.

## Workflow

### 1. Gather what you need (ask if it's not already in the conversation)

- Who's traveling, who's hosting (if anyone) — first names are enough
- Destination(s) and date range
- **Exact arrival and departure date+time, and which timezone they're written in.** This is the single most common source of errors — see the pitfall below before you hardcode anything.
- The day-by-day plan, if it's already been discussed — pull it from the conversation rather than re-asking
- Any home base, private address, or meeting point that will appear on the page
- Theme preference, if the user has one — otherwise don't ask, build one from the destination (see step 3)

Don't block on having every detail. Draft with placeholders (`[fill in]`) for anything missing and flag them clearly in your reply, rather than stalling the whole page on one unknown.

### 2. Privacy pass — do this before writing a single address

**If the traveler is staying in a hotel, Airbnb, or other named lodging**, it's fine to write the property name and neighborhood plainly — it's already a public business, and naming it is genuinely useful (for a rideshare driver, for a friend meeting up). Just leave out the room/unit number.

**If the traveler is staying at someone's private home** (a friend's or family member's place), never write the literal address, exact building name, or precise residential neighborhood into the file — even if the user gave it to you directly in chat — if the page might ever be screenshotted, shared, hosted at a guessable URL, or turned public.

Pattern that works well for the private-home case: replace the specific location with something like *"the address I sent you"* or *"our usual spot"* — it reads as a natural inside-joke between the people who actually need the info, and reveals nothing to an outside reader. Keep transit line names and general area ("a station on the X line") since that's genuinely useful and not identifying on its own.

Apply this consistently: if a specific place name appears in more than one spot (e.g. a home neighborhood mentioned both in an "essentials" card AND inside a route description like "take the train to X"), sanitize *all* of them the same way. A single redacted card next to three casual mentions of the real name defeats the point.

### 3. Build a theme from the destination

Every trip has a destination, so every page gets a theme drawn from that destination. Don't ask the user to pick one and don't ship a generic default — a page themed to where they're actually going is the difference between a document and something they want to open.

The two themes in the template are **starting points to copy, not a menu**: `kawaii-pink` (warm, soft, decorative) and `neutral-modern` (restrained, typographic). Pick whichever is closer in temperature to the destination, then retheme it.

The theme is one attribute:

```html
<html data-theme="lisbon-tile">
```

Everything else — fonts, colors, the decorative header shape — follows automatically from that one value; you don't need to touch the rest of the CSS. To build a destination theme: copy the `[data-theme="..."]` CSS block nearest in feel, rename it after the place, and adjust the color variables.

Pull three or four colors from something the place is actually known for — a material, a landscape, a building tradition, a light quality. Lisbon's azulejo blue and terracotta. Kyoto's temple vermilion against cedar. Reykjavík's slate and moss. Look for what someone who's been there would recognize, not the flag colors and not the first stock photo.

`examples/demo-lisbon-trip.html` is exactly this, done once — a `lisbon-tile` theme built by copying a base block and swapping the variables.

Two ways this goes wrong:

- **Overdoing it.** The theme lives in the palette and one or two decorative elements. Don't add clip art, emoji rows, or a second font per destination — legibility on a phone at 8am beats atmosphere.
- **Reaching too hard.** If a place resists an obvious palette, theme the *trip* instead — a winter city break, a coastal week, a work trip with weekends attached. That's still specific to them, and it beats a forced cliché.

Only override this if the user asks for a specific theme, or asks for one of the two base themes by name.

The header already includes a gradient background with a soft dot texture, a wave-shaped divider into the page below it, a stat "chips" row, and an optional decorative corner accent (`.corner-orb`) and small icon (like `.bow` for kawaii-pink) that each theme can turn on or leave off — you don't need to rebuild any of this, just fill in the chip text and swap which decorative elements show for a new theme (see how `.bow`/`.tile-ornament`/`.corner-orb` are scoped per `data-theme` near the top of the CSS).

### 4. Build the day cards

Each day is a `<section class="day">` with a `data-date` (ISO format, used by the nav-builder script) and a `data-title` (short label shown in the tab). Inside, a header with a colored badge, then a `.timeline` of `.item` blocks: a time, a bolded title, one or two sentences, and optional tags.

Tags available out of the box: `free`, `book` (needs advance booking), `price` (put the actual cost in the tag text, e.g. "¥500" or "$12" — currency varies by trip), `transit` (a transport instruction worth calling out), `solo` (fine to do solo / a solo-friendly window), `highlight` (a generic accent tag for whatever the trip's personal touch is — a fandom, an inside joke, a "don't miss this" flag). Add new tag classes the same way if the trip needs a category these don't cover.

Keep description text to one or two sentences per item — this is a glanceable phone reference, not a blog post.

### 5. Wire up the countdown — read this before setting the dates

Near the bottom of the file:

```js
var arrive = new Date('2026-07-18T15:05:00+09:00');
var depart = new Date('2026-07-25T18:00:00+09:00');
```

The header shows a live, ticking Days/Hours/Min/Sec countdown before the trip starts, then switches automatically to "Day N of the trip" during it, then "Trip complete" after — this is handled by the `tick()` function running every second, you don't need to build this part.

**Common mistake to actively check for:** a booking confirmation's departure date is the date the traveler *leaves the origin city*, not the date they land. On a long-haul or overnight flight, local arrival time is very often the *next calendar day* at the destination — itineraries mark this with a small "+1 day" next to the arrival time, which is easy to miss. Before hardcoding the arrival timestamp, explicitly confirm: departure date + flight duration, checked against the destination timezone offset, actually lands on the date you're about to write down. Getting this wrong means the whole countdown (and the day-nav's "today" highlighting) is off by a day, usually discovered only when someone is waiting at an airport for a flight that isn't landing yet.

Also double-check the UTC offset in the ISO string matches the *destination's* timezone, not the traveler's home timezone.

### 6. Sticky day nav

This is built automatically from whatever `.day` sections exist — you don't need to write it by hand. Two behaviors worth knowing about, both already implemented:
- The active tab is determined by scroll position and highlighted as the user scrolls
- The tab strip **auto-scrolls itself horizontally** to keep the active tab visible. Without this, a user scrolling past day 6 or 7 finds the corresponding tab has scrolled off the visible edge of the nav bar and has to scroll it manually — this was a real bug in an earlier version, the fix is the `scrollTo` block inside `syncActive()`. Don't remove it when customizing.

### 7. Fill in the checklist and essentials panels

- **Checklist** (`#checklist`): bookings and prep tasks, each a checkbox. Note in the UI copy that checkboxes reset on reload (there's no backend) — don't let the user think it's persistent.
- **Essentials**: where they're staying (see the privacy pass above for hotel vs. private-home wording), a meeting point if separated, emergency numbers *for the destination country* (don't default to one country's numbers), a weather/heat-or-cold plan, cash/payment norms, and anything else that's genuinely useful in a moment of minor chaos (a delayed train, a dead phone). This panel should stay short — five or six cards, not a full guidebook.

### 8. Before you deliver, check

- [ ] Arrival/departure dates and timezones are correct (re-read step 5)
- [ ] No literal home address or exact private location anywhere in the file, including inside route descriptions, not just the essentials card
- [ ] The theme is drawn from the destination (or from what the user explicitly asked for), not left on a base theme
- [ ] Nav auto-scroll still works if you modified the script
- [ ] Checkboxes visibly cross out their label when checked
- [ ] Save to the output/deliverable location and present the file to the user (use whichever file-delivery tool is available in this environment; if none exists, tell the user the file path directly)

### 9. Expect to keep editing it

Trip plans change constantly — delayed flights, bad weather, a fully-booked restaurant, a swapped day. When the user comes back with a change, edit the specific section with a targeted replace rather than regenerating the whole file; the user may have already customized copy elsewhere that a full rewrite would silently discard. Re-deploying (if hosted) is just re-uploading the same file — remind the user to do this after edits if they mentioned a host like Vercel or Netlify, since edits made here don't push themselves.

## Files in this skill

- `assets/template.html` — the starting point. Copy it, then follow the steps above.
