# bossmade-events — Enhancement Plan
**Last updated:** 2026-04-10
**Site:** https://bossmade-events.vercel.app
**GitHub:** github.com/MathiasFobi/bossmade-events

---

## ✅ Already Done (2026-04-10)

- [x] Event images on all 15 cards (Unsplash by category)
- [x] `data-cat`, `data-search`, `data-day`, `data-id` attrs on event cards
- [x] Debounced search (200ms)
- [x] Quiz `window.events` fix
- [x] "This Weekend" / "Next Week" filter pills
- [x] Category count badges on filter pills
- [x] Smooth tab fade animation
- [x] Back-to-top floating button
- [x] dates.json validUntil extended to Apr 26
- [x] Remove Date Night tab (unused, always expired)
- [x] Remove Quiz tab (broken, depends on dates.json)

---

## 🔴 High Priority

### 1. Ticket Purchase Links
**Problem:** 4 events show "Varies" for price with no way to buy tickets.
**Fix:** Add a `ticketUrl` field to events.json for all events. Add a "🎟 Get Tickets" button on each card that opens the URL in a new tab.

```json
// events.json field
{ "ticketUrl": "https://www.eventbrite.com/..." }
```

```js
// In buildEventCard, add:
${ev.ticketUrl ? `<a class="card-btn" href="${ev.ticketUrl}" target="_blank" rel="noopener">🎟 Tickets</a>` : ''}
```

### 2. Mistral API Key Exposure
**Problem:** `xN26eRj8gZjHi19JPyCefeJIVfYGDHlk` hardcoded in index.html — anyone can View Source and steal it.
**Fix:** Move the Page Agent to a Cloudflare Worker proxy. The Worker holds the key server-side and forwards messages to Mistral. Frontend only sees the Worker URL.

```
User → index.html → Cloudflare Worker (has API key) → Mistral → Worker → User
```

### 3. Price Indicator System
**Problem:** "Varies" on 4/15 events is unhelpful for users deciding whether to attend.
**Fix:** Add a `priceLevel` field: `$` (under $15), `$$` ($15-$50), `$$$` (over $50), `Free`. Display as emoji: 💚 / 💛 / 💜 / 🟢 Free in the card meta.

---

## 🟡 Medium Priority

### 4. Google Fonts + Typography Polish
**Problem:** Site uses system fonts. Could feel more premium.
**Fix:** Add Inter or Space Grotesk from Google Fonts. Update `--font` in CSS.

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
body { font-family: 'Inter', sans-serif; }
```

### 5. Share to Instagram/Stories
**Problem:** Currently only Twitter/X share works, and it shares the page URL not the event.
**Fix:** Add WhatsApp share (deep link: `https://wa.me/?text=...`), Instagram Stories share (via Web Share API with `text` param), and copy-event-link that copies an event-specific deep link.

### 6. Venue Parking / Transit Info
**Problem:** Big venues (Centennial Olympic Park, Cobb Energy Centre) have no parking guidance.
**Fix:** Add `venueNotes` field: `"MARTA: Arts Center station"`, `"Free parking on site"`, `"No parking — use rideshare"`. Display as a small info line under location.

### 7. Past Events Section
**Problem:** Events that have passed disappear completely. No archive.
**Fix:** After the main "Coming Up" section, add a collapsed "Past Events" accordion that shows the last 3-5 events that have passed. Keeps the page clean but preserves history.

### 8. Event Details Modal
**Problem:** Clicking an event card does nothing — user has to mentally parse all info from the card.
**Fix:** Add a click handler on the card title that opens a modal with: full description, map embed (Google Maps link), ticket button, share buttons, save button. Slide-up modal with backdrop blur.

---

## 🟢 Nice to Have

### 9. "Happening Tonight" Hero Banner
**Problem:** The most urgent event (Fernbank tonight) is buried in the feed.
**Fix:** If there are events happening today, show a full-width hero banner above the search bar: "🔥 TONIGHT: Fernbank After Dark — 7 PM — $24 — 🎟 Get Tickets". Drives immediate action.

### 10. Age Rating Badges
**Problem:** Fernbank After Dark is 21+, Dogwood is family-friendly — no way to filter.
**Fix:** Add `ageRating` field: "🧸 All Ages", "👶 Kids Under 10", "🔞 21+". Add filter pills for age rating.

### 11. Sold Out / Limited Tickets Badge
**Problem:** No event ever shows as sold out, even when it is.
**Fix:** Add `soldOut: false` and `ticketsRemaining: N` fields. Show "❌ Sold Out" or "🎟 Few Left!" badge on cards when applicable.

### 12. Auto-Refresh / Live Updates
**Problem:** User has to manually refresh to see new events from the cron job.
**Fix:** Add a "Last updated X minutes ago" counter. Poll `meta.json` every 5 minutes. If `generated` timestamp changes, silently refresh the event list with a subtle flash animation. No full page reload.

### 13. DM to Book / Text Reminder
**Problem:** Users find an event but forget to follow through.
**Fix:** Add a "🔔 Remind Me" button that opens a small form: phone number + "Send me a reminder 2 hours before". Store phone in localStorage. On remind trigger, show a toast: "We'll text you before the event!" (Integrates with Twilio or a Cloudflare Worker.)

### 14. Multi-Day Event Grouping
**Problem:** Dogwood Festival (3 entries) and SweetWater 420 (2 entries) are separate cards for what should be one event.
**Fix:** Add `groupId` field to events.json. Events with the same `groupId` render as one card with a day-selector tab strip: "Day 1 (Apr 10) | Day 2 (Apr 11) | Day 3 (Apr 12)".

---

## 📊 Potential Enhancements

### A. Weather Integration
Show weather forecast for the event day/event location. "🌤️ 72°F and sunny — perfect for Dogwood Festival!" Use wttr.in free API.

### B. "Perfect For..." Tags
Add auto-generated secondary tags: `#DateNight`, `#SoloFriendly`, `#FamilyVibes`, `#LateNight`, `#Outdoor`. Derive from category + description keywords.

### C. Animated Empty State
When search/filter returns 0 results, show a subtle animated illustration (CSS only) instead of just text.

### D. Keyboard Navigation
Power users can tab through cards, press Enter to open modal, Escape to close modal. Accessibility score improvement.

---

## 🚫 Out of Scope

- Adding a full backend (DB, auth) — static site is a feature, not a bug
- Payment processing — ticket links go to external platforms
- Comments/reviews — too complex for v1

---

## Technical Debt

1. **Git author identity** — commits show "Personal Assistant" instead of BossMade
2. **No `alt` text on decorative elements** — improve accessibility
3. **No error boundary on fetch()** — network failures show cryptic empty state
4. **dates.json** — still in repo but unused; remove to reduce confusion
5. **buildEventCard uses template literals** — safe for now but sanitize `ev.title` if user-generated content is ever added
