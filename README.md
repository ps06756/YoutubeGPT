# YouTube Video Summarizer

A Chrome extension that summarizes YouTube videos directly on the page using Claude AI. It extracts the video transcript and generates a structured summary in a convenient sidebar.

## Features

- **Sidebar on YouTube** — A collapsible right sidebar appears on any YouTube video page
- **Transcript-based** — Uses the video's existing captions/subtitles (no audio processing)
- **No login required** — Just provide your own Anthropic API key
- **Privacy-first** — Your API key is stored locally in the browser, never sent to any server other than Anthropic's API
- **SPA-aware** — Works seamlessly when navigating between YouTube videos without page reloads
- **Dark theme** — Matches YouTube's dark interface

## Installation

1. Clone or download this repository
2. Open Chrome and navigate to `chrome://extensions/`
3. Enable **Developer mode** (toggle in the top-right corner)
4. Click **Load unpacked** and select the project folder
5. The extension icon will appear in your Chrome toolbar

## Usage

1. Navigate to any YouTube video
2. Click the floating toggle button (bottom-right area of the page) to open the sidebar
3. Enter your [Anthropic API key](https://console.anthropic.com/settings/keys) and click **Save**
4. Click **Summarize This Video** to generate a summary

The summary includes:
- **Overview** — A brief 2-3 sentence summary
- **Key Points** — Bullet points of the main ideas
- **Takeaways** — Actionable or notable takeaways

You can also manage your API key from the extension popup (click the extension icon in the toolbar).

## Requirements

- Google Chrome (or any Chromium-based browser)
- An [Anthropic API key](https://console.anthropic.com/settings/keys)
- The YouTube video must have captions/subtitles available (most videos have auto-generated captions)

## How It Works

1. The content script runs on YouTube video pages
2. When you click "Summarize", it fetches the video's caption track data
3. The transcript text is sent to the Anthropic Claude API (claude-sonnet-4-20250514)
4. The structured summary is rendered in the sidebar

## Privacy

- Your Anthropic API key is stored in `chrome.storage.local` (browser-local, never leaves your machine)
- The only external request is to `api.anthropic.com` to generate the summary
- No analytics, no tracking, no data collection
- The video transcript is sent directly to Anthropic's API — no intermediary servers

## Limitations

- Videos without captions/subtitles cannot be summarized
- Very long transcripts (100k+ characters) are truncated to keep API costs reasonable
- Auto-generated captions may contain inaccuracies that affect summary quality

## License

[GNU Affero General Public License v3.0](LICENSE)
