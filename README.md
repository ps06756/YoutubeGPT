# YoutubeGPT

A Chrome extension that summarizes YouTube videos directly on the page using AI. It extracts the video transcript and generates a structured summary in a convenient sidebar.

## Features

- **Sidebar on YouTube** — A collapsible right sidebar appears on any YouTube video page
- **Transcript-based** — Uses the video's existing captions/subtitles (no audio processing)
- **No login required** — Just provide your own API key for your preferred AI provider
- **Privacy-first** — Your API key is stored locally in the browser, never sent to any server other than your chosen AI provider
- **SPA-aware** — Works seamlessly when navigating between YouTube videos without page reloads
- **Dark theme** — Matches YouTube's dark interface
- **Multi-provider support** — Works with Anthropic, OpenAI, and Google Gemini

## Supported AI Providers

Choose your preferred AI provider:

- **Anthropic** — Claude Sonnet 4, Claude Haiku 4.5
- **OpenAI** — GPT-4o, GPT-4o Mini, and any OpenAI-compatible API (e.g., Kimi K2.5)
- **Google Gemini** — Gemini 2.5 Flash, Gemini 2.5 Pro

## Installation

1. Clone or download this repository
2. Open Chrome and navigate to `chrome://extensions/`
3. Enable **Developer mode** (toggle in the top-right corner)
4. Click **Load unpacked** and select the project folder
5. The extension icon will appear in your Chrome toolbar

## Usage

1. Navigate to any YouTube video
2. Click the floating toggle button (bottom-right area of the page) to open the sidebar
3. Click the extension icon in the toolbar to open settings
4. Select your preferred AI provider and model
5. Enter your API key and click **Save**
6. Click **Summarize This Video** to generate a summary

The summary includes:
- **Overview** — A brief 2-3 sentence summary
- **Key Points** — Bullet points of the main ideas
- **Takeaways** — Actionable or notable takeaways

You can also manage your API key from the extension popup (click the extension icon in the toolbar).

## Requirements

- Google Chrome (or any Chromium-based browser)
- An API key from one of the supported providers:
  - [Anthropic API key](https://console.anthropic.com/settings/keys)
  - [OpenAI API key](https://platform.openai.com/api-keys)
  - [Google Gemini API key](https://aistudio.google.com/app/apikey)
- The YouTube video must have captions/subtitles available (most videos have auto-generated captions)

## How It Works

1. The content script runs on YouTube video pages
2. When you click "Summarize", it fetches the video's caption track data
3. The transcript text is sent to your selected AI provider's API
4. The structured summary is rendered in the sidebar

## Privacy

- Your API key is stored in `chrome.storage.local` (browser-local, never leaves your machine)
- The only external request is to your chosen AI provider's API to generate the summary
- No analytics, no tracking, no data collection
- The video transcript is sent directly to the AI provider — no intermediary servers

## Limitations

- Videos without captions/subtitles cannot be summarized
- Very long transcripts (100k+ characters) are truncated to keep API costs reasonable
- Auto-generated captions may contain inaccuracies that affect summary quality

## License

[GNU Affero General Public License v3.0](LICENSE)
