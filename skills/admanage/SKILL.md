---
name: admanage
description: >-
  Launch Meta/TikTok/Snapchat/Pinterest ads, query analytics (spend, CPA, ROAS), browse Google Drive and Dropbox for creatives, manage campaigns, and upload media — all via the AdManage MCP tools. Use when the user wants to launch ads, check ad performance, find creatives, or manage their ad accounts.
allowed-tools: Bash, Read, Write, Grep, Glob, WebFetch, mcp__admanage__get_current_user, mcp__admanage__get_launch_defaults, mcp__admanage__list_ad_accounts, mcp__admanage__list_workspaces, mcp__admanage__list_profiles, mcp__admanage__list_campaigns, mcp__admanage__list_adsets, mcp__admanage__launch_ads, mcp__admanage__get_batch_status, mcp__admanage__list_batches, mcp__admanage__get_batch, mcp__admanage__list_drafts, mcp__admanage__create_draft, mcp__admanage__launch_draft, mcp__admanage__query_reports, mcp__admanage__get_report_fields, mcp__admanage__get_top_ads, mcp__admanage__get_daily_spend, mcp__admanage__search_media, mcp__admanage__get_media, mcp__admanage__upload_media_from_url, mcp__admanage__upload_media_from_file, mcp__admanage__list_library_assets, mcp__admanage__get_library_asset, mcp__admanage__list_library_boards, mcp__admanage__list_board_assets, mcp__admanage__list_library_tags, mcp__admanage__browse_google_drive, mcp__admanage__browse_dropbox, mcp__admanage__duplicate_campaign, mcp__admanage__duplicate_adset, mcp__admanage__duplicate_ad, mcp__admanage__list_ad_copy_templates, mcp__admanage__get_ad_copy_template, mcp__admanage__list_automations, mcp__admanage__get_automation
argument-hint: "launch a Meta ad", "show my top ads this month", "browse Dropbox for creatives"
---

# AdManage — Launch Ads & Query Analytics from Claude

You have access to 36 AdManage MCP tools. Use them to help the user launch ads, analyze performance, browse media, and manage campaigns.

## Quick Launch Workflow (Most Common)

When the user wants to launch an ad, follow this exact flow:

### Step 1: Get defaults and context

Call these in parallel to save time:

- `get_launch_defaults` — returns saved page, Instagram, ad copy, CTA, link, UTM tags
- `get_current_user` — returns user info, default workspace, default ad account

This gives you everything needed to pre-fill the launch. Only ask the user for what's missing.

### Step 2: Get ad sets

- `list_adsets` with the user's ad account ID and a limit of 10-20
- Present the ad sets and let the user pick, or auto-select if they specified one

### Step 3: Get media

Media can come from multiple sources:

- **User provides a URL** — use directly in the `media` array
- **User drags a file into chat (Cowork)** — the MCP server is remote and can't access local files. Try IN ORDER:
  1. **Direct upload** (best): `curl -X POST "https://api.admanage.ai/v1/media/upload" -H "Authorization: Bearer API_KEY" -F "file=@/path/to/file"` — returns media.admanage.ai URL directly
  2. **Litterbox** (fallback): `curl -F "reqtype=fileupload" -F "time=24h" -F "fileToUpload=@/path/to/file" https://litterbox.catbox.moe/resources/internals/api.php` → `upload_media_from_url`
  3. **Public URL / cloud**: `upload_media_from_url`, `browse_dropbox`, `browse_google_drive`
- **User drags a file into chat (Claude Code)** — use `upload_media_from_file` with the filePath directly
- **Google Drive** — use `browse_google_drive` to find files, use the `webContentLink` or `downloadUrl`
- **Dropbox** — use `browse_dropbox` to list files, each has a `launchUrl` ready for launching
- **AdManage Library** — use `list_library_assets` or `search_media` to find existing creatives
- **Dropbox shared link** — pass the full URL as `sharedLink` to `browse_dropbox`

### Step 4: Launch

Call `launch_ads` with:

```json
{
  "adAccountId": "act_123...",
  "adSets": [{ "value": "12345", "label": "US Broad" }],
  "media": [{ "url": "https://...", "name": "creative.mp4", "type": "video" }],
  "title": "from defaults",
  "description": "from defaults",
  "cta": "from defaults",
  "link": "from defaults",
  "page": "from defaults",
  "insta": "from defaults"
}
```

### Step 5: Poll status

Call `get_batch_status` with the returned `adBatchId`. Poll up to 3 times (5s apart) until status is `success` or `error`.

Report to the user:
- Status (success/error)
- Number of ads created
- Ads Manager link (in the response as `adsManagerUrl`)
- AdManage link (in the response as `admanageUrl`)
- Any errors with details

## Ad Formats

| Format | `type` | When to use |
|--------|--------|-------------|
| Single | `"single"` | 1 image or video per ad (default) |
| Multi-placement | `"multi"` | 2+ format variants (1:1, 9:16) — platform picks best per placement |
| Carousel | `"carousel"` | 2+ cards with per-card `carouselTitle`, `carouselDescription`, `carouselLink` |
| Flexible | `"flexible"` | Advantage+ with `headlineVariations[]` and `bodyVariations[]` |

## Analytics Workflow

When the user asks about performance:

1. `get_current_user` — get their default ad account
2. `query_reports` — flexible BigQuery query with metrics, groupBy, date range
3. Or use shortcuts: `get_top_ads`, `get_daily_spend`

Common metrics: `spend`, `impressions`, `clicks`, `ctr`, `cpc`, `cpa`, `conversions`, `roas`
Common groupBy: `ad`, `adset`, `campaign`, `day`, `platform`

## Media from Cloud Storage

### Google Drive
```
browse_google_drive(folderId: "root")  // list root
browse_google_drive(folderId: "1ABc...", search: "spring")  // search in folder
```

### Dropbox
```
browse_dropbox(path: "")  // list root
browse_dropbox(path: "/Marketing/Creatives")  // subfolder
browse_dropbox(sharedLink: "https://www.dropbox.com/scl/fo/abc123/...")  // shared folder
```

Each Dropbox file returns a `launchUrl` — pass it directly as the media URL when launching.

## Important Rules

1. **Always get defaults first** — don't ask the user for page/insta/copy if defaults exist
2. **Always poll batch status** — don't just fire-and-forget the launch
3. **Use parallel tool calls** — call `get_launch_defaults` + `get_current_user` simultaneously
4. **Show Ads Manager links** — always include the link so users can verify in Facebook/TikTok
5. **Handle errors gracefully** — if a launch fails, show the error and suggest fixes
6. **businessId required for list_profiles** — always pass the ad account ID to avoid huge responses
7. **Media type detection** — mp4/mov/avi = video, png/jpg/gif = image. The API auto-detects from URL extension
8. **Multiple ads in one call** — you can pass multiple objects in the `ads` array to launch several ads at once

## Platform-Specific Fields

### TikTok
- `selectedTikTokUserAccount` — required for TikTok launches

### Snapchat
- `snapchatBrandName`, `snapchatProfileId`, `snapchatAdType`, `snapchatHeadline`, `snapchatCTA`

### Pinterest
- `pinterestBoardId`, `pinterestCTA`

## API Documentation

Full endpoint docs: https://api.admanage.ai/llms.txt
Get your API key: https://admanage.ai/connect
