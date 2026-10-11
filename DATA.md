# Plan Maadi data files

The website reads two files from this folder:

| File | What's in it | Who updates it |
|---|---|---|
| `events.json` | Everything that's on: events, plus the metro stations, extra food spots and road-distance table the planner uses | The scraper, rebuilt every day |
| `festivals.json` | Festival specials: dates, the best places to go, timings, tips and sources | Kept by hand and topped up by the scraper. **Merged, never overwritten.** |
| `food.json` | Restaurants, cafés and eateries the planner uses for meal stops | Researched from Reddit and Bengaluru food guides; re-checked by hand or scraper. **Merged, never overwritten.** |

`festivals.json` and `food.json` are optional. If either is missing or broken, the site still works, just without festival specials or with only its built-in eateries.

---

## festivals.json

From two weeks before a festival (or from `showFrom`) until its last day, if the festival has at least one place, the site adds a **"Festival: <name>" button** to "What do you feel like?", showing its dates. Picking it:

- moves the dates to the festival if they were outside it
- shows a short line of context (on for these days only, `about`, and a "Good to know" fold with `tips` and `sources`)
- asks "Which day?" (the `days`), plus the festival's own follow-up questions (`facets`)
- plans with the festival's places, plus any listing in `events.json` for it. A listing belongs to the festival if the scraper sets `"festival": "<festival id>"` on it, if its title matches one of the festival's `keywords`, or if its category is in the festival's `cats`. For example, Navratri takes in the "navratri" garba and dandiya listings, and while it's on, the separate "Navratri and garba" button steps aside.

Up to two festival buttons show at once: the one with the higher `priority` first, otherwise the soonest. A festival marked `"draft": true` is in the calendar but never shown. Use that for future festivals whose places haven't been checked for that year yet.

**To preview a date:** open the site with `?debug&today=2026-12-20` and it behaves as if it's that day.

```jsonc
{
  "updated": "2026-10-10",
  "festivals": [
    {
      "id": "durga-puja-2026",            // unique, with the year
      "name": "Durga Puja",
      "chip": "Festival: Durga Puja",      // the button in "What do you feel like?" (default "Festival: " + name)
      "color": "#B3261E",                  // accent for its follow-up questions
      "cats": ["navratri"],                // optional: listing categories the festival takes in
      "cuisine": ["bengali", "odia"],      // optional: meals on a festival day prefer these cuisines (see food.json)
      "foodKey": "durga-puja",             // optional: ...and restaurants whose "festivals" include this key
      "veg": true,                         // optional: prefer pure-veg places for its meals
      "facets": [                          // optional follow-up questions, shown after "Which day?"
        { "id": "see", "q": "What would you like?", "opts": [
          { "id": "sindoor", "label": "Sindoor khela", "match": "sindoor|sindur", "on": "rituals" }, // regex on that day's rituals only
          { "id": "bhog", "label": "Bhog and Bengali food", "match": "bhog|food|anandamela" },      // regex on title, venue, about, food or that day's rituals
          { "id": "big", "label": "The biggest pujas", "field": "big" }                             // places with "big": 1
        ] }
      ],
      "from": "2026-10-17", "to": "2026-10-21",
      "showFrom": "2026-10-03",            // optional; defaults to 14 days before "from"
      "status": "confirmed",               // or "expected" while the date isn't announced (the site says "expected date")
      "draft": false,                      // true = in the calendar only, not shown yet
      "priority": 1,                       // optional; higher shows first when two festivals overlap
      "days": [ { "date": "2026-10-19", "label": "Ashtami" } ],
      "about": "One or two plain sentences of context, shown when the festival is picked.",
      "keywords": ["durga puja", "pujo", "pandal", "dhunuchi", "sindoor khela"],
      "tips": ["Go after 9 pm on Ashtami to skip the worst crowds."],
      "sources": [ { "name": "Deccan Herald", "url": "https://…", "date": "2026-10-08" } ],
      "checked": "2026-10-10",             // last time anyone checked the festival's details
      "places": [
        {
          "id": "bengalee-association-ulsoor",   // unique within the festival
          "t": "Bengalee Association Durga Puja",
          "venue": "Bengalee Association", "area": "Ulsoor",
          "ll": [12.9822, 77.6203],             // required; places without it are skipped
          "hrs": [8, 23],                        // required; the hours the planner may use, 24-hour clock (past midnight = 24+)
          "hoursText": "Hours not published yet", // what visitors see instead of the hours above, e.g. when the real hours aren't known
          "dates": ["2026-10-17", "2026-10-18"], // optional; defaults to every day of the festival
          "h": 1,                                // how long a visit takes, in hours (default 1)
          "p": 0,                                // entry fee per person: 0 = free, null = not known
          "out": 1,                              // 1 if it's outdoors
          "big": 1,                              // optional flag a facet can use ("field": "big")
          "about": "Short factual highlight.",
          "rituals": [ { "date": "2026-10-19", "time": "8:00 am", "what": "Ashtami anjali" } ], // "date": "daily" for every day; "time" can be ""
          "food": "Bhog after the morning anjali; food stalls in the evenings.",
          "tags": ["date", "friends", "solo", "social", "family", "elders", "kids"], // who it suits (default: everyone)
          "x": { "sm": "n", "pe": "u", "ki": "y", "wc": "u", "lo": "y" },        // smoking, pets, kids, wheelchair, loud: y / n / p / u
          "u": "https://source-or-organiser-page",
          "src": "Deccan Herald",
          "checked": "2026-10-10",
          "year_of_info": 2026,
          "curated": true                        // checked by a person: the scraper must not change or remove it
        }
      ]
    }
  ]
}
```

### Rules for the scraper

1. **Merge, don't replace.** Match festivals and places by `id`. Update only the fields you actually scraped and keep everything else. Never delete a festival, and never change or remove a place marked `"curated": true`.
2. **Keep the calendar a year ahead.** Every major festival in the next 12 months should have an entry with `from` and `to` (`"status": "expected"` until the date is announced). A festival without places stays invisible on the site, so the calendar is safe to fill early.
3. **Start collecting about three weeks before** each festival, from **this year's** sources only: news (Deccan Herald, The Hindu, Times of India, Bangalore Mirror, The News Minute), organisers' own websites, Facebook and Instagram pages, and listings (HighApe, District, LBB, allevents). Put `u`, `src`, `checked` and `year_of_info` on every place.
4. **Re-check every day while it's on.** Timings and ritual times change at short notice. Update `checked` on the festival each run, even when nothing changed.
5. **Don't carry old information forward.** A place whose `year_of_info` is before last year should be dropped, unless a person marked it `curated`. When you've refreshed a draft festival's places for this year, set `"draft": false` so it starts showing.
6. **Tag festival events** in `events.json` with `"festival": "<festival id>"` when you're sure. Otherwise the site falls back to `keywords`, so keep those specific: "durga puja", not just "puja".
7. **Stay in the city.** The site doesn't plan out-of-town trips, so skip places outside Bengaluru (e.g. Mysuru Dasara).

---

## food.json

Meal stops (breakfast, lunch, coffee, dinner, late-night bites) are chosen from these, near the plan's other stops. The choice depends on:

- **The occasion:** places to linger for a date or with elders; family places for kids; quick and easy with friends.
- **The festival, if one is picked:** its `cuisine` and `foodKey`.
- **Variety:** not the same cuisine twice in a day.
- **Budget, constraints and opening days.**

```jsonc
{
  "updated": "2026-10-10",
  "restaurants": [
    {
      "id": "bhojohori-manna-koramangala",
      "name": "Bhojohori Manna", "area": "Koramangala", "address": "…",
      "ll": [12.93, 77.62],                       // required; within ~200 m of the door
      "cuisines": ["bengali"],                    // from the list in the research notes: south-indian, udupi, karnataka, bengali, …
      "festivals": ["durga-puja", "poila-baishakh"], // optional; matches a festival's "foodKey"
      "veg_only": false, "alcohol": false,
      "cost_pp": [400, 700],                      // required; rupees per person
      "meals": ["lunch", "dinner"],               // breakfast, lunch, coffee, dinner, late
      "good_for": ["family", "elders", "iconic"], // date, family, kids, elders, friends, groups, solo, quick, iconic, fine-dining, rooftop, outdoor, …
      "known_for": "Kosha mangsho, bhetki paturi",
      "closed_days": [1],                         // 0 = Sunday … 6 = Saturday, or null
      "u": "https://source", "src": "Reddit r/bangalore", "checked": "2026-10-10",
      "status": "open: 2026 listing"              // how we know it's still open
    }
  ]
}
```

**Rules:** match by `id` and merge. Re-check `status` every few months, because Bengaluru restaurants close often. Drop a place you can't confirm is open, but never silently change one a person has checked (`"curated": true`).

---

## events.json (summary)

`updated`, `window` ([first day, last day]), `sources` (names shown in the footer), and:

- `events`: one object per listing: `t` title, `dates` (YYYY-MM-DD list), `time` (text as listed), `venue`, `area`, `cat` (category, e.g. comedy, music, workshop, food), `p` lowest price (null if not shown), `tags` (who it suits, plus alc / veg / outdoor / night), `src`, `u` booking link, `note`, `rec` (regular / recurring), `ll` [lat, lng], `id`, `s` time slots, `h` hours, `d` (1 = out of town; these are skipped), `x` venue facts, and optionally `festival`
- `spots`: extra food and drink places (name, area, ll, cost range, slots, tags, attributes)
- `attrs`: venue facts for built-in places, keyed by name
- `metro`: stations with name, line, lat, lng
- `roads`: the pre-computed road distance and drive-time table between known points
- `highlights` (optional): up to three event ids to show in "What's the hype this week" instead of the automatic picks
