# ChatGPT Conversation Exporter

> A Chrome extension that exports **all your ChatGPT conversations** to a single flat JSON file — preserving project metadata, with no Python or external tools required.

[![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Manifest V3](https://img.shields.io/badge/Manifest-V3-orange.svg)](https://developer.chrome.com/docs/extensions/mv3/intro/)

---

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Output Format](#output-format)
- [How It Works](#how-it-works)
- [Privacy](#privacy)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Features

- **Exports all projects** — automatically discovers every project in your sidebar
- **Flat JSON output** — one file, all conversations, ready for analysis or further processing
- **Project metadata preserved** — each conversation includes its project name (`null` for non-project chats)
- **Exports regular conversations too** — non-project chats are included with `"project": null`
- **Runs in the background** — close the popup while exporting; the tab keeps working
- **Live progress + ETA** — real-time progress bar with time-remaining estimate
- **No Python, no scripts, no dependencies** — pure JavaScript, one JSON file download
- **Zero external requests** — uses your existing browser session; nothing leaves your machine

---

## Installation

### Option A — Load unpacked (no store required)

1. [Download this repository](https://github.com/vincze-tamas/chatgpt-exporter/archive/refs/heads/main.zip) and unzip it
2. Open Chrome and go to `chrome://extensions`
3. Enable **Developer mode** (toggle in the top-right corner)
4. Click **Load unpacked** and select the `chatgpt-exporter/` folder

The extension icon appears in your toolbar. If it's hidden, click the puzzle-piece icon and pin *ChatGPT Conversation Exporter*.

---

## Usage

### Step 1 — Open ChatGPT

Navigate to [chatgpt.com](https://chatgpt.com) and make sure you are logged in.

### Step 2 — Load all your projects

In the left sidebar, hover over the **"More"** button under *Projects*. This makes ChatGPT render all your project links, which the extension picks up automatically.

> If only some projects appear, wait a moment or scroll the sidebar — the extension detects them as they appear in the DOM.

### Step 3 — Export

Click the **ChatGPT Exporter** icon in your toolbar, then click **📥 Export to JSON**.

The extension will:
1. Detect all your projects
2. Download the conversation list for each project
3. Download full conversation details and flatten them to JSON
4. Download `chatgpt_export_YYYY-MM-DD.json` automatically

> **You can close the popup** while the export runs — it continues in the background as long as the `chatgpt.com` tab stays open. Reopen the popup anytime to check progress.

---

## Output Format

The exported file is a JSON array where each element represents one conversation:

```json
[
  {
    "project": "My Coding Project",
    "conversation_id": "a1b2c3d4-...",
    "title": "Refactor authentication module",
    "created_at": "2025-06-01T10:15:00.000Z",
    "messages": [
      { "role": "user", "content": "Can you help me refactor this JWT middleware?" },
      { "role": "assistant", "content": "Sure! Here's a cleaner version..." }
    ]
  },
  {
    "project": null,
    "conversation_id": "e5f6g7h8-...",
    "title": "Recipe ideas",
    "created_at": "2025-07-10T08:00:00.000Z",
    "messages": [
      { "role": "user", "content": "What can I make with zucchini?" },
      { "role": "assistant", "content": "Here are a few ideas..." }
    ]
  }
]
```

Conversations not belonging to any project have `"project": null`.

---

## How It Works

ChatGPT projects are stored internally as **gizmos** with the URL pattern `/g/g-p-{hex_id}-{slug}/`. The extension:

1. Watches the DOM with a `MutationObserver` to capture project links as they appear
2. Optionally clicks "More" in the sidebar to reveal all projects
3. Fetches a session token from `/api/auth/session` (same-origin, no password required)
4. Retrieves conversation lists via `/backend-api/gizmos/{gizmo_id}/conversations?cursor=0`
5. Uses cursor-based pagination to collect every conversation across all pages
6. Fetches full conversation trees via `/backend-api/conversation/{id}`
7. Flattens each conversation tree to a `messages` array (`role` + `content` only)
8. Serialises everything to a single JSON file and triggers a browser download

All API calls are made **from within the `chatgpt.com` page context** using your existing browser session — identical to what the ChatGPT web app does itself.

---

## Privacy

This extension:

- **Only runs on `chatgpt.com`** — it cannot access any other website
- **Makes no external network requests** — all requests go to `chatgpt.com` only
- **Sends no data anywhere** — the JSON is built in memory and saved directly to your computer
- **Uses no third-party libraries** — the full codebase is ~300 lines of vanilla JavaScript
- **Stores nothing persistently** — no `localStorage`, no `IndexedDB`, no cookies written

The full source code is auditable in this repository.

---

## Requirements

- Chrome 109+ or any Chromium-based browser (Edge, Brave, Arc, …)
- A ChatGPT account (Free or Plus)

---

## Troubleshooting

| Symptom | Solution |
|---|---|
| "Reload the chatgpt.com tab and try again" | The content script didn't load. Reload the `chatgpt.com` tab, then open the popup again. |
| 0 projects detected | Hover over "More" in the sidebar first, wait 1–2 seconds, then click Export. |
| Export stops mid-way | Keep the `chatgpt.com` tab open and visible. The extension keeps running in the background. |
| JSON file is empty or very small | A temporary API error occurred — try again. |
| Conversations appear under the wrong project | Make sure you're on the latest version; reload the extension if needed. |

**Still stuck?** [Open an issue](https://github.com/vincze-tamas/chatgpt-exporter/issues) and include:
- Your Chrome version (`chrome://version`)
- What you see in DevTools Console (`F12` → Console tab, on the `chatgpt.com` tab)
- Approximate number of projects and conversations

---

## Contributing

Contributions are welcome!

- **Bug reports & feature requests** → use the [issue templates](.github/ISSUE_TEMPLATE/)
- **Code changes** → fork, branch, and open a pull request with a clear description
- **Security issues** → see [SECURITY.md](SECURITY.md) before posting publicly

---

## License

[MIT](LICENSE) — free to use, modify, and distribute.
