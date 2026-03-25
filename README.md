# AdManage Skills for Claude Code

Launch Meta, TikTok, Snapchat, and Pinterest ads, query analytics, browse media from Google Drive and Dropbox -- all from Claude Code.

## Setup

### 1. Get your API key

Go to [AdManage Settings > API Keys](https://admanage.ai/settings) and generate an API key (starts with `ak_`).

### 2. Add the MCP server

```bash
claude mcp add --transport http admanage https://mcp.admanage.ai/mcp \
  --header "Authorization: Bearer ak_YOUR_API_KEY"
```

Replace `ak_YOUR_API_KEY` with your actual API key from step 1.

### 3. Install the skill

```bash
claude skill add --from https://github.com/Admanage-ai/admanage-skills
```

Or copy `skills/admanage/SKILL.md` into your project's `.claude/skills/admanage/` directory.

## Skills

### `/admanage` - Launch & Manage Ads

The main skill for launching ads, querying performance, and managing campaigns across Meta, TikTok, Snapchat, and Pinterest.

**What it can do:**
- Launch single, multi-placement, carousel, and flexible ads
- Use saved defaults (page, Instagram, copy, CTA, link) for quick launches
- Query analytics: spend, CPA, ROAS, CTR, impressions
- Browse media from Google Drive, Dropbox, and the AdManage library
- Upload local files to your media library
- Manage drafts, templates, and automations
- Duplicate campaigns, ad sets, and ads

**Example prompts:**
- "Launch this video as a Meta ad to my US Broad ad set"
- "Show me my top 5 ads by spend this month"
- "Browse my Dropbox for new creatives and launch them"
- "What's my daily spend for the last 7 days?"
- "Duplicate my best performing campaign"

## 36 MCP Tools

| Category | Tools |
|----------|-------|
| **Account & Setup** | `get_current_user`, `get_launch_defaults`, `list_ad_accounts`, `list_workspaces`, `list_profiles` |
| **Campaigns** | `list_campaigns`, `list_adsets` |
| **Launching** | `launch_ads`, `get_batch_status`, `list_batches`, `get_batch` |
| **Drafts** | `list_drafts`, `create_draft`, `launch_draft` |
| **Analytics** | `query_reports`, `get_report_fields`, `get_top_ads`, `get_daily_spend` |
| **Media** | `search_media`, `get_media`, `upload_media_from_url`, `upload_media_from_file` |
| **Library** | `list_library_assets`, `get_library_asset`, `list_library_boards`, `list_board_assets`, `list_library_tags` |
| **Cloud Storage** | `browse_google_drive`, `browse_dropbox` |
| **Management** | `duplicate_campaign`, `duplicate_adset`, `duplicate_ad` |
| **Templates** | `list_ad_copy_templates`, `get_ad_copy_template` |
| **Automations** | `list_automations`, `get_automation` |

## Links

- [AdManage](https://admanage.ai) - Ad management platform
- [API Docs](https://api.admanage.ai/llms.txt) - Full API documentation
- [Privacy Policy](https://admanage.ai/privacy) - How we handle your data
- [Support](mailto:support@admanage.ai) - Get help
