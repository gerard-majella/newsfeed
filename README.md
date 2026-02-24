# Industry Newsfeed Dashboard

A lightweight, client-side newsfeed dashboard that aggregates articles by industry and deploys automatically to GitHub Pages.

---

## What is this?

This repository hosts a single-page static dashboard (`index.html`) that reads from `data.json` and renders industry-tabbed article cards. There is no build step and no backend — everything runs in the browser.

The feed is intended to be updated weekly by an external automation tool (e.g. [n8n](https://n8n.io)) via the GitHub Contents API, which writes a new `data.json` directly to this repository.

---

## How `data.json` is structured

```json
{
  "updated": "2025-02-24T00:00:00Z",
  "industries": [
    {
      "id": "manufacturing",
      "label": "Manufacturing",
      "description": "Keywords: industrial, manufacturing, ICS",
      "articles": [
        {
          "title": "Article Title",
          "source": "Source Name",
          "date": "2025-02-24",
          "url": "https://example.com/article",
          "summary": "A brief summary of the article.",
          "tags": ["ICS", "Manufacturing"]
        }
      ]
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `updated` | ISO 8601 string | Timestamp shown in the dashboard header as "Week of …" |
| `industries` | array | One entry per industry tab |
| `industries[].id` | string | Unique slug used as HTML `id` |
| `industries[].label` | string | Human-readable tab label |
| `industries[].description` | string | Short description shown above the cards |
| `industries[].articles` | array | Articles for that industry |
| `articles[].title` | string | Article headline |
| `articles[].source` | string | Publication / source name |
| `articles[].date` | string | Publication date (`YYYY-MM-DD`) |
| `articles[].url` | string | Link to the original article |
| `articles[].summary` | string | One- or two-sentence summary |
| `articles[].tags` | string[] | Keyword tags displayed as pills |

---

## How to update `data.json` via the GitHub Contents API

External tools (e.g. n8n) can update `data.json` with a single **PUT** request:

### 1. Get the current file SHA (required for updates)

```http
GET https://api.github.com/repos/{owner}/{repo}/contents/data.json
Authorization: Bearer {GITHUB_TOKEN}
```

Parse the `sha` field from the response.

### 2. PUT the new content

```http
PUT https://api.github.com/repos/{owner}/{repo}/contents/data.json
Authorization: Bearer {GITHUB_TOKEN}
Content-Type: application/json

{
  "message": "chore: update newsfeed data",
  "sha": "{current_sha}",
  "content": "{base64_encoded_json}"
}
```

- Replace `{owner}` and `{repo}` with the repository owner and name.
- Replace `{GITHUB_TOKEN}` with a fine-grained personal access token that has **Contents: Read & Write** permission for this repository.
- `content` must be the **base64-encoded** string of the full `data.json` payload.

The dashboard fetches `data.json` with a cache-busting timestamp query parameter (`?_=<timestamp>`) so users always see the latest data without a hard refresh.

---

## Local development

Open `index.html` directly in a browser. If `data.json` cannot be fetched (e.g. when using the `file://` protocol), the dashboard automatically falls back to a built-in sample payload so you can still preview the UI.

For a proper local test, serve the directory with any static file server:

```bash
npx serve .
# or
python3 -m http.server
```

---

## Deployment

The repository deploys automatically to **GitHub Pages** via the workflow at `.github/workflows/pages.yml` on every push to `main`.
