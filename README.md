# MTG Art Browser

A single-file, browser-based tool for browsing Magic: The Gathering card artworks, building a 9-slot print sheet, and exporting it as a PDF for proxy printing.

**No backend, no build step, no server required.** Open `index.html` directly in any modern browser.

---

## Features

### Search
- **Scryfall search** — finds all unique printings of a card by exact name, including promos, showcase, borderless, extended art, and Secret Lair variants (`unique=prints` + `include:extras` query)
- **DeviantArt search** — searches DeviantArt via RSS for fan art / alternate proxies (`mtg <query>`)
- **Batch search** — paste a list of card names (one per line), searches all on Scryfall sequentially with progress and cancel

### Results (left panel)
- All results from both sources appear in a **combined scrollable thumbnail grid**
- Each card shows its source badge (SC / DA) and a label on hover (set, year, artist, frame effects)
- Click a thumbnail to **preview it large** on the right — it does not immediately add to the sheet

### Preview (right panel)
- Selected artwork is shown large with card name, label, and source badge
- **"+ Add to Sheet"** — adds to the first empty slot in the print sheet, removes thumbnail from grid, and saves the file locally
- **"⬇ Download"** — always triggers a direct browser download to the Downloads folder (independent of folder picker state)
- **"Open ↗"** link (DeviantArt items only) — opens the original deviation page

### Print Sheet (bottom panel)
- 9-slot 3×3 grid
- Drag and drop local image files directly onto any slot
- Click a filled slot to remove it
- **"📁 Set folder"** — picks a local directory (Chrome/Edge) for auto-saving clicked artworks; Firefox falls back to browser download automatically
- **"Generate PDF"** — exports a US Letter PDF (3×3 grid, 10mm margins, 3mm gaps, ~63×84mm cells per card)
- Empty slots render as grey outline placeholders in the PDF

### Layout
- **Resizable panels** — drag the divider between top (search/results) and bottom (sheet) sections
- Results container is split: left = small thumbnail grid, right = large preview pane

---

## Technical Details

### Scryfall API
- Endpoint: `GET https://api.scryfall.com/cards/search`
- Query: `!"<card name>" include:extras` with `unique=prints`, `order=released`
- CORS-enabled — works directly from browser, no proxy needed
- 100ms delay per request (rate limit compliance)
- Double-faced cards: uses `card.card_faces[0].image_uris.normal`
- Client-side URL dedup via `Set` — avoids showing truly identical images

### DeviantArt
- RSS feed: `https://backend.deviantart.com/rss.xml?q=mtg+<query>&type=deviation&limit=24`
- Routed through `corsproxy.io` for CORS
- Parsed with `DOMParser` using `getElementsByTagNameNS('http://search.yahoo.com/mrss/', 'content')`

### Local File Saving
- **Chrome/Edge**: File System Access API (`showDirectoryPicker`) — saves directly to a chosen folder
- **Firefox**: Falls back to `<a download>` trigger — saves to configured Downloads folder
- Filename format: `<card_name>_<8hex_hash>.jpg` (hash derived from image URL for uniqueness)

### PDF Generation
- Library: jsPDF 2.5.1 UMD (loaded from CDN)
- Format: US Letter portrait (215.9 × 279.4 mm)
- Layout: 10mm margins, 3mm gaps → cells ~63.3mm × 84.5mm
- Images scaled to fit with MTG card ratio (5:7), centered in each cell
- Uses canvas + `crossOrigin='anonymous'` to convert image URLs to data URLs for jsPDF
- Local (drag-dropped) images stored as base64 data URLs — passed directly to jsPDF

### Drag & Drop
- Each sheet slot accepts dropped image files (`image/*`)
- `FileReader.readAsDataURL()` stores them as base64 in state
- `dragleave` flicker fix: `slot.contains(e.relatedTarget)` check

---

## Current State (as of last session)

### What works
- Scryfall search with all printing variants (showcase, extended art, promos, borderless, etc.)
- DeviantArt fan art search via RSS proxy
- Batch Scryfall search with progress/cancel
- Left thumbnail grid + right large preview panel
- "Add to Sheet", "Download", "Open ↗" buttons in preview
- Drag & drop local images onto sheet slots
- Folder picker (Chrome) / auto-download fallback (Firefox)
- Resizable top/bottom panels via drag handle
- PDF generation (US Letter, 3×3 grid)
- Removing a card from the sheet by clicking it

### Known issues / limitations
- DeviantArt RSS results are general fan art — not always card proxies
- DeviantArt images may occasionally fail to load in PDF (CORS on some wixmp CDN URLs)
- No pagination for Scryfall results (limited to first page ~175 results)
- `corsproxy.io` is a free public proxy and may be rate-limited or go down

### Potential next steps
- Add pagination / "load more" for Scryfall results
- Filter/sort controls for the results grid (by set, year, artist)
- Drag thumbnails from grid directly onto sheet slots (instead of only the preview → Add flow)
- Search history / recent cards
- Right-click context menu on thumbnails
- Additional art sources (e.g. MTG Goldfish, Moxfield proxies)

---

## File Structure

```
mtg-art-browser/
├── index.html    ← entire app (HTML + CSS + JS, ~1100 lines)
└── artworks/     ← local saved images (gitignored)
```

All logic is inline in `index.html`. No build system, no dependencies except jsPDF from CDN.
