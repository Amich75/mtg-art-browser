# MTG Art Browser

A single-file, browser-based tool for browsing Magic: The Gathering card artworks, building an unlimited print sheet, and exporting it as a multi-page PDF for proxy printing.

**No backend, no build step, no server required.** Open `index.html` directly in any modern browser.

**Current version: v0.2**

---

## Features

### Search
- **Scryfall** — finds all unique printings of a card by exact name, including promos, showcase, borderless, extended art, and Secret Lair variants (`unique=prints` + `include:extras` query)
- **MPC Fill** — searches [mpcfill.com](https://mpcfill.com) for proxy card images from the community
- Both sources are **toggled independently** — enable/disable either with the source buttons next to the search bar
- **Paste List** — always-visible textarea below the search bar; paste a list of card names (plain or Moxfield export format), searches all enabled sources sequentially with progress and cancel

### Paste List (bulk search)
- Plain card names, one per line: `Lightning Bolt`
- Moxfield export format: `1 Black Lotus (LEA) 232` or `1 Counterspell (2ED) 61 *F*`
- Searches both Scryfall and MPC Fill in parallel per card (based on enabled source toggles)
- Progress indicator and Cancel button during search

### Results (left panel — 80% width)
- Results are **grouped by card name** — each card gets its own section with a header, result count, and a per-card **"✕ Clear"** button to remove just that card's results
- Each card shows its source badge (SC / MPC) and a label on hover
- **"✕ Clear"** button in the panel header clears all results and resets the preview
- **Single click** a thumbnail to **preview it large** on the right
- **Double click** a thumbnail to **add it directly to the print sheet** (skips the preview step)

### Preview (right panel — 20% width)
- Selected artwork shown large with card name, label, and source badge
- **"+ Add to PDF / Double Click"** — appends to the print sheet, removes thumbnail from grid, saves locally
- **"⬇ Download"** — direct browser download (independent of folder picker)
- **"Open ↗"** link — opens the original source page (where available)

### Print Sheet (bottom panel)
- **Unlimited slots** — add as many cards as you want, no 9-card limit
- Scrollable 3-column grid; each slot maintains the MTG card aspect ratio (5:7)
- Drag and drop local image files directly onto any empty slot
- Click a filled slot to remove it
- **"📁 Set folder"** — picks a local directory (Chrome/Edge) for auto-saving artworks; Firefox falls back to browser download
- **PDF counters** — two counters next to the Generate PDF button show total PDF pages and current page occupancy (x/9)
- **"Generate PDF"** — exports a US Letter PDF with a 3×3 grid per page; generates as many pages as needed
- **"Clear"** — removes all cards from the sheet

### Layout
- **Resizable panels** — drag the divider between the top (search/results) and bottom (sheet) sections
- Left panel = scrollable thumbnail grid (80%), right panel = large preview (20%)

---

## Technical Details

### Scryfall API
- Endpoint: `GET https://api.scryfall.com/cards/search`
- Query: `!"<card name>" include:extras` with `unique=prints`, `order=released`
- CORS-enabled — works directly from browser, no proxy needed
- 100ms delay per request (rate limit compliance)
- Double-faced cards: uses `card.card_faces[0].image_uris.normal`

### MPC Fill API
- Endpoint: `POST https://mpcfill.com/2/exploreSearch/` (via `corsproxy.io`)
- Sources fetched from `GET /2/sources/` on first search and cached for the session
- Images hosted on Google Drive — no CORS headers, so `crossOrigin` attribute is never set for MPC Fill images
- Fetch for PDF/download routed through `corsproxy.io` as well

### CORS Handling
- **Scryfall**: native CORS support — `crossOrigin="anonymous"` set on images, canvas used for PDF
- **MPC Fill**: no CORS headers from Google Drive — `crossOrigin` attribute never set; PDF uses `fetch → blob → FileReader` via proxy

### Local File Saving
- **Chrome/Edge**: File System Access API (`showDirectoryPicker`) — saves directly to chosen folder
- **Firefox**: Falls back to `<a download>` trigger — saves to configured Downloads folder
- Filename format: `<card_name>_<8hex_hash>.jpg`

### PDF Generation
- Library: jsPDF 2.5.1 UMD (loaded from CDN)
- Format: US Letter portrait (215.9 × 279.4 mm)
- Layout: 10mm margins, 3mm gaps → cells ~63.3mm × 84.5mm
- Cards chunked 9 per page; `doc.addPage()` called for each additional page
- Scryfall images: canvas + `crossOrigin='anonymous'`
- MPC Fill images: `fetch → blob → FileReader` through CORS proxy

### Drag & Drop
- Trailing empty slots always present for dropping local image files
- `FileReader.readAsDataURL()` stores images as base64 in state
- `dragleave` flicker fix: `slot.contains(e.relatedTarget)` check

---

## Known Issues / Limitations
- No pagination for Scryfall results (limited to first page, ~175 results)
- MPC Fill images may occasionally fail in PDF if Google Drive throttles the proxy
- `corsproxy.io` is a free public proxy and may be rate-limited or go down

## Potential Next Steps
- Add pagination / "load more" for Scryfall results
- Filter/sort controls for results grid (by set, year, artist)
- Drag thumbnails directly from grid onto sheet slots
- Search history / recent cards
- Additional art sources

---

## File Structure

```
mtg-art-browser/
├── index.html    ← entire app (HTML + CSS + JS, ~1375 lines)
└── artworks/     ← local saved images (gitignored)
```

All logic is inline in `index.html`. No build system, no dependencies except jsPDF from CDN.
