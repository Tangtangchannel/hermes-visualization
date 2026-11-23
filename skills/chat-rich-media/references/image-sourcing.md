# Sourcing and verifying images for a display

## Search: Wikimedia Commons first

```python
p = {"action":"query", "format":"json", "generator":"search",
     "gsrsearch": f"{query} filetype:bitmap", "gsrnamespace":"6", "gsrlimit":"8",
     "prop":"imageinfo", "iiprop":"url|size|mime|extmetadata", "iiurlwidth":"960"}
# GET https://commons.wikimedia.org/w/api.php?<urlencoded p>
# header User-Agent: something identifiable — anonymous bursts get throttled
```

Read `extmetadata.LicenseShortName` for every hit and keep it for the caption.

## Download

- Prefer the **`thumb.wikimedia.org`** URL from `imageinfo.thumburl`. The unscaled
  `upload.wikimedia.org` origin answers `HTTP 429 Too many requests` after a couple of rapid hits, and
  keeps doing it for a while.
- Retry with backoff (3s, then 6s, then 9s) and treat a 429 as "slow down", not "missing".
- The API appends `?utm_source=…&utm_campaign=…`; **strip the query string** before pasting a URL into
  markdown or HTML.
- Sanity gate every download: `status == 200`, `Content-Type: image/*`, bytes over ~40KB, natural width
  at least 600px.

## Is it the right subject?

Weakest to strongest evidence:

1. `vision_analyze` on the downloaded file — a single model pass.
2. Commons file description and categories.
3. `globalusage` — how many wikis embed the file (`prop=globalusage`). A portrait used by ~20 language
   versions of one person's article is that person.
4. The article's own canonical image:
   `https://<lang>.wikipedia.org/api/rest_v1/page/summary/<Title>` → `originalimage.source`.

Rules that came out of getting this wrong:

- **A vision pass is not an identity check.** It has confidently named two *different* public figures for two
  files, and was wrong on both; faces in formal settings are where it fails most. Never let one vision
  pass veto a Wikipedia-canonical file, and never ship a photo on vision alone.
- **Trust the encyclopedia over the filename.** Search results are user-uploaded; a file named after a
  person can be someone else entirely, and a "headquarters" query can return only interior showrooms.
- **Caption what is in the frame**, not what you hoped it showed: "exhibition hall, network open-platform
  booth" is honest; "corporate headquarters" is not, and the reader can see it.

## Reject on sight

Wrong person, watermarks or agency stamps, collages, screenshots of social posts, anything under ~600px,
AI-generated "portraits", and images whose license is unclear.

## Display

- Remote markdown images (`![](url)`) render inline but depend on hotlinking and can 429 later. For
  anything the reader will keep, download next to the widget (`~/.hermes/widgets/img/`) and reference it
  relatively, with the remote URL as an `onerror` fallback.
- Print the license on the tile (`CC BY-SA 4.0`) and one attribution line under the gallery. Creative
  Commons licenses require attribution and share-alike; commercial press photos do not belong in a
  deliverable at all.
- Crop for consistency: fixed tile height, `object-fit: cover`, `object-position: center top` for
  portraits. A wall of mismatched aspect ratios reads as scraped rather than curated.
