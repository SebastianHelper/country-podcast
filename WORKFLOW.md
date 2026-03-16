# Country Podcast Workflow

## 1. Country Order (By Region)

Proposed regional grouping with ~195 countries:

| Region | Countries | Order |
|--------|-----------|-------|
| Europe | ~44 | 1-44 |
| Asia | ~48 | 45-92 |
| Middle East | ~17 | 93-109 |
| Africa | ~54 | 110-163 |
| North America | ~23 | 164-186 |
| South America | ~12 | 187-198 |
| Oceania | ~14 | 199-212 |

Within each region: alphabetical by common English name.

**Alternative:** Group by sub-region first (Western Europe → Eastern Europe → Central Asia → East Asia → etc.) for better geographic flow.

---

## 2. Research Pipeline

### Step 1: Fetch Wikipedia Content

**Do NOT rely on model knowledge.** Always fetch fresh from Wikipedia.

**Sources:**
- Primary: `https://en.wikipedia.org/wiki/{Country_name}`
- Secondary: `https://en.wikipedia.org/wiki/History_of_{Country_name}` (for deeper history)
- Current events: `https://en.wikipedia.org/wiki/Politics_of_{Country_name}`

**API Options:**
```
# Summary (quick overview)
https://en.wikipedia.org/api/rest_v1/page/summary/{title}

# Full article HTML
https://en.wikipedia.org/api/rest_v1/page/html/{title}

# Raw wikitext (for parsing)
https://en.wikipedia.org/w/index.php?title={title}&action=raw
```

**Recommended approach:** Use `web_fetch` on the main article, then extract sections:
- History
- Politics / Government
- Geography
- Demographics
- International relations / Conflicts

### Step 2: Extract Relevant Content

Parse the Wikipedia article and extract:
1. **History section** — key events, founding, major conflicts
2. **Politics section** — current government type, leader, stability
3. **International relations** — conflicts with neighbors, alliances
4. **Geography context** — location, neighbors, key features

### Step 3: Generate Script

Use LLM to synthesize extracted content into a ~15-minute audio script:
- Target: ~2000-2500 words (at ~150 words/minute = 15 min)
- Structure:
  1. Introduction (where is it, neighbors)
  2. Historical overview (key events)
  3. Current political situation
  4. Regional context / conflicts
  5. Closing summary

**Prompt template:**
```
You are writing a podcast script about [Country].
Use ONLY the following Wikipedia content as source.
Do not add information not present in the source.

[Wikipedia content here]

Write a 15-minute podcast script covering:
1. Geographic location and neighbors
2. Key historical events
3. Current political situation
4. Regional conflicts and relations
5. Brief summary

Target length: 2000-2500 words.
Tone: Informative, engaging, neutral.

TTS-FRIENDLINESS RULES:
- Write for spoken delivery, not reading
- Use short, clear sentences (avoid nested clauses)
- Avoid abbreviations (write "United States" not "US")
- Spell out numbers under 100 ("twenty-three" not "23")
- Avoid URLs, file paths, code, or technical notation
- No markdown formatting (no **bold**, no headers, no lists)
- Use natural transitions between topics ("Now let's look at...", "Moving on to...")
- Include brief pauses as paragraph breaks (TTS will naturally pause)
- Avoid foreign words without pronunciation guide, or anglicize them
- No footnotes, citations, or references in the spoken text
```

### Step 4: Convert to Audio

Use existing `md-to-audio.py` utility:
```bash
python3 /data/.openclaw/workspace/utilities/md-to-audio.py script.md output.mp3
```

### Step 5: Generate Map

**OpenStreetMap options:**

1. **Static Map API (recommended):**
   ```
   https://staticmap.openstreetmap.de/staticmap.php?center={lat},{lon}&zoom={z}&size=600x400
   ```
   Need country coordinates (can get from Wikipedia infobox).

2. **Python `staticmap` library:**
   ```python
   from staticmap import StaticMap, Polygon
   # Draw country outline from GeoJSON
   ```

3. **Browser screenshot:**
   - Navigate to OpenStreetMap country page
   - Screenshot the map area

**Best approach:** Use country bounding box from Natural Earth or GeoJSON, render via `staticmap` library.

---

## 3. GitHub Pages Hosting

**Yes, audio files can be hosted on GitHub Pages.**

### Setup

1. Create a dedicated repo: `country-podcast`
2. Structure:
   ```
   /audio/
     /2026/
       /03/
         afghanistan.mp3
         albania.mp3
         ...
   /maps/
     /2026/
       /03/
         afghanistan.png
         ...
   index.html  # optional listing page
   ```
3. Enable GitHub Pages in repo settings (source: main branch)
4. Audio accessible at: `https://tibaerius.github.io/country-podcast/audio/2026/03/afghanistan.mp3`

### Limitations

- **File size limit:** 100MB per file (GitHub's soft limit)
- **Bandwidth:** 100GB/month soft limit (plenty for personal use)
- **Build time:** Static files served immediately, no build needed

### Alternative: GitHub Releases

For larger files or better organization:
- Create a release for each country (or batch)
- Attach audio + map as release assets
- No size limit per se, but 2GB per release

---

## 4. Full Pipeline (Pseudocode)

```python
def generate_country_podcast(country_name: str, date: str):
    # 1. Fetch Wikipedia
    wiki_url = f"https://en.wikipedia.org/wiki/{country_name}"
    wiki_content = web_fetch(wiki_url)

    # 2. Extract sections
    history = extract_section(wiki_content, "History")
    politics = extract_section(wiki_content, "Politics")
    relations = extract_section(wiki_content, "Foreign relations")

    # 3. Generate script
    script = llm_generate_script(history, politics, relations)

    # 4. Convert to audio
    audio_file = md_to_audio(script, f"{country_name}.mp3")

    # 5. Generate map
    coords = get_country_coords(country_name)  # from Wikipedia infobox
    map_file = generate_map(coords, f"{country_name}.png")

    # 6. Deliver
    return audio_file, map_file
```

---

## 5. Next Steps

1. ~~**Prototype with one country** (e.g., Afghanistan or Albania)~~ → **Switzerland selected**
2. **Test Wikipedia fetching** via `web_fetch`
3. **Test map generation** with OpenStreetMap static API
4. **Set up GitHub repo** for hosting
5. **Decide on scheduling** (cron vs heartbeat)

### Prototype: Switzerland

- **Status:** ✅ Complete
- **Started:** 2026-03-15
- **Completed:** 2026-03-15
- **Files:**
  - `switzerland-source.md` — Wikipedia source material
  - `switzerland-script.md` — TTS-friendly podcast script
  - `switzerland.mp3` — Audio file (9:09 duration, 3.2 MB)
  - `switzerland-map.png` — Map image (512x512, OpenStreetMap tiles)

---

## 6. Open Questions

- [x] Which country to start with for prototype? → **Switzerland completed**
- [ ] Should we include a "This day in history" segment?
- [ ] Voice preference for TTS? (default: en-US-ChristopherNeural)
- [ ] Should maps show country highlighted or just location marker?
- [ ] How to handle countries with ongoing conflicts (sensitivity)?

## 7. TTS Engine Testing

See `BACKLOG.md` for full comparison. Options to test:

| Engine | Quality | Cost | Status |
|--------|---------|------|--------|
| Edge TTS (current) | ⭐⭐⭐ | Free | ✅ Baseline |
| Piper TTS | ⭐⭐⭐ | Free | 📋 To test |
| OpenAI TTS | ⭐⭐⭐⭐ | $0.18/ep | 📋 Needs API key |
| ElevenLabs | ⭐⭐⭐⭐⭐ | $5/mo | 📋 Trial available |

**Test plan:**
1. Set up Piper TTS Docker on VPS
2. Generate test audio with Switzerland script
3. Compare quality with Edge TTS
4. If insufficient, evaluate OpenAI TTS