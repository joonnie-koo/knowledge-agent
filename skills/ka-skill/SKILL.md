---
name: ka-skill
description: AI-powered knowledge management system. Search books, save to Notion, get recommendations, and record reviews via Telegram.
homepage: https://github.com/yourusername/knowledge-agent
metadata: {"clawdbot":{"emoji":"📚"}}
---

# knowledge-agent

Intelligent knowledge management through AI. Store and organize books, articles, and papers with automatic metadata enrichment.

Setup (once)
- Set environment variables for API keys:
  - `GEMINI_API_KEY` - Google Gemini API key
  - `BRAVE_SEARCH_API_KEY` - Brave Search API key
  - `NOTION_API_KEY` - Notion integration token
  - `GOOGLE_CHAT_WEBHOOK_URL` - Google Chat webhook URL

Common commands
- Search book info: `ka search-book --title "Book Title"`
- Save to Notion: `ka save --title "Title" --author "Author"`
- Get recommendations: `ka recommend --count 5`
- Record review: `ka review --title "Book Title" --content "My thoughts..."`

Notes
- All API keys must be configured before first use
- Supports books, articles, and papers (extensible)
- Google Chat is the primary input/output channel
