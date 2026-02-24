# How to Fetch YouTube Transcripts (No API Key)

Reverse-engineered from "YouTube Summary with ChatGPT & Claude" extension v2.0.23.

---

## Method 1 — Internal JSON API (Preferred)

**Endpoint:**
```
POST https://www.youtube.com/youtubei/v1/get_transcript?prettyPrint=false
```

**Request body** (standard YouTube InnerTube context):
```json
{
  "context": {
    "client": {
      "clientName": "WEB",
      "clientVersion": "2.20240101"
    }
  },
  "params": "<VIDEO_ID_OR_TRANSCRIPT_PARAMS_TOKEN>"
}
```

**Response path to transcript segments:**
```
actions[0]
  .updateEngagementPanelAction
  .content.transcriptRenderer
  .content.transcriptSearchPanelRenderer
  .body.transcriptSegmentListRenderer
  .initialSegments[]
```

Each segment has `startMs`, `endMs`, and the text.

---

## Method 2 — Caption XML URL (Fallback)

### Step 1: Get the caption URL from the page HTML

Every YouTube video page contains `ytInitialPlayerResponse` embedded in a `<script>` tag.
Fetch the page HTML and extract caption tracks:

```js
async function getCaptionTracks(videoId) {
  const html = await fetch(`https://www.youtube.com/watch?v=${videoId}`).then(r => r.text());
  const split = html.split('"captions":');
  if (split.length < 2) return []; // video has no captions
  const captionsJson = JSON.parse(
    split[1].split(',"videoDetails"')[0]
  );
  return captionsJson.playerCaptionsTracklistRenderer.captionTracks;
  // Each track: { baseUrl, name: { simpleText }, vssId, languageCode }
}
```

### Step 2: Fetch the caption XML

```js
async function fetchCaptionXml(track, potToken = null) {
  let url = track.baseUrl;
  if (potToken) url += `&pot=${potToken}&c=WEB`;

  const xml = await fetch(url).then(r => r.text());
  // Parse XML — each <text start="1.23" dur="4.5">Hello world</text>
  return xml;
}
```

### Step 3: Parse XML into segments

```js
function parseXml(xmlText) {
  // Using DOMParser in browser, or node-html-parser in Node.js
  const doc = new DOMParser().parseFromString(xmlText, "text/xml");
  return Array.from(doc.getElementsByTagName("text")).map(node => ({
    start: parseFloat(node.getAttribute("start")),
    duration: parseFloat(node.getAttribute("dur")),
    text: node.textContent
      .replace(/&#(\d+);/g, (_, code) => String.fromCharCode(code)) // decode HTML entities
      .replace(/<[^>]*>/g, "")                                        // strip any tags
      .trim()
  }));
}
```

---

## Getting the `pot` Token (if needed)

Some videos require a `pot` (proof-of-origin token) appended to the caption URL.

**In a browser extension / content script context:**
```js
async function getPotToken(videoId) {
  // Check cache first
  const cached = localStorage.getItem(`yt-caption-potoken-${videoId}`);
  if (cached) return cached;

  // Click YouTube's own transcript button and intercept the network request
  const btn = document.querySelector('button[aria-label*="transcript" i]');
  if (!btn) return null;

  return new Promise(resolve => {
    btn.addEventListener("click", async () => {
      performance.clearResourceTimings();
      for (let i = 0; i <= 500; i += 50) {
        await new Promise(r => setTimeout(r, 50));
        const req = performance
          .getEntriesByType("resource")
          .filter(r => r.name.includes("/api/timedtext?"))
          .pop();
        if (req) {
          const pot = new URL(req.name).searchParams.get("pot");
          if (pot) {
            localStorage.setItem(`yt-caption-potoken-${videoId}`, pot);
            resolve(pot);
            return;
          }
        }
      }
      resolve(null);
    }, { once: true });
    btn.click();
  });
}
```

> **Note:** The `pot` token is only needed in browser contexts with certain videos.
> For server-side scraping, Method 1 (JSON API) typically doesn't require it.

---

## Full Working Example (Node.js / Server-side)

```js
async function getYouTubeTranscript(videoId, lang = "en") {
  // 1. Fetch page HTML
  const html = await fetch(`https://www.youtube.com/watch?v=${videoId}`, {
    headers: { "Accept-Language": "en-US,en;q=0.9" }
  }).then(r => r.text());

  // 2. Try Method 1 first — internal JSON API
  try {
    const paramsMatch = html.match(/"getTranscriptEndpoint":\{"params":"([^"]+)"/);
    if (paramsMatch) {
      const res = await fetch(
        "https://www.youtube.com/youtubei/v1/get_transcript?prettyPrint=false",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            context: { client: { clientName: "WEB", clientVersion: "2.20240101" } },
            params: paramsMatch[1]
          })
        }
      ).then(r => r.json());

      const segments = res
        .actions?.[0]
        ?.updateEngagementPanelAction
        ?.content?.transcriptRenderer
        ?.content?.transcriptSearchPanelRenderer
        ?.body?.transcriptSegmentListRenderer
        ?.initialSegments ?? [];

      return segments.map(s => ({
        start: s.transcriptSegmentRenderer?.startMs / 1000,
        text: s.transcriptSegmentRenderer?.snippet?.runs?.[0]?.text ?? ""
      })).filter(s => s.text);
    }
  } catch {}

  // 3. Fallback — caption XML
  const captionsSplit = html.split('"captions":');
  if (captionsSplit.length < 2) throw new Error("No captions found");

  const tracks = JSON.parse(
    captionsSplit[1].split(',"videoDetails"')[0]
  ).playerCaptionsTracklistRenderer.captionTracks;

  // Pick preferred language
  const track = tracks.find(t => t.languageCode === lang) ?? tracks[0];
  if (!track) throw new Error("No matching language track");

  const xml = await fetch(track.baseUrl).then(r => r.text());

  // Parse XML
  const matches = [...xml.matchAll(/<text start="([^"]+)" dur="([^"]+)"[^>]*>([\s\S]*?)<\/text>/g)];
  return matches.map(([, start, dur, text]) => ({
    start: parseFloat(start),
    duration: parseFloat(dur),
    text: text
      .replace(/&#(\d+);/g, (_, c) => String.fromCharCode(c))
      .replace(/<[^>]*>/g, "")
      .trim()
  }));
}

// Usage
const transcript = await getYouTubeTranscript("dQw4w9WgXcQ");
console.log(transcript.map(s => s.text).join(" "));
```

---

## Quick Reference

| | Method 1 (JSON API) | Method 2 (XML) |
|--|--|--|
| Endpoint | `youtubei/v1/get_transcript` | `baseUrl` from page HTML |
| Auth needed | No | Sometimes needs `pot` token |
| Works server-side | Yes | Yes (without `pot`) |
| Response format | JSON | XML |
| Availability | Newer videos | Most videos |

---

## Important Notes

- **No YouTube Data API key required** for either method
- YouTube may throttle or block repeated requests from the same IP — add delays if scraping at scale
- `baseUrl` from `ytInitialPlayerResponse` expires; always re-fetch for fresh URLs
- Auto-generated captions (`vssId` starting with `a.`) are lower quality than manual ones
- The `pot` token approach only works in a browser/extension context, not server-side
