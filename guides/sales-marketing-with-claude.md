# Sales & Marketing with Claude Code

How we use Claude Code beyond engineering — for CRM, prospecting, content, and social media at Proportione.

## The stack

A 1-3 person consultancy can't afford dedicated marketing tools at enterprise prices. Our approach: **lifetime AppSumo deals + Claude Code as glue**.

| Area | Tools | Claude's role |
|------|-------|---------------|
| CRM | Stackby (SoT), LeadCRM, Skarbe | Built MCP server for CRM access from terminal |
| Email marketing | Acumbamail, Brevo, SendFox | Generates drafts, follow-ups via CRM MCP |
| SEO | Semdash, ClickRank, Search Console | Analyzes Search Console data via BI MCP |
| Analytics | GA4, Looker Studio, Growify | GA4 reports accessible via BI MCP |
| Content | Pixelied, Minvo, Depositphotos | Claude generates editorial calendars and copy |
| Automation | Pabbly Connect, Google Apps Script | Claude generates rules; Pabbly executes |
| Browser automation | BrowserAct | Scraping and web automation with AI prompts |
| Social scheduling | Radaar.io | Content pool + scheduler across 5 channels |

**Monthly recurring cost**: ~200-250 EUR (mostly hosting + Google Workspace + Claude Max)
**Lifetime tools**: 30+ AppSumo deals covering 80% of needs

## CRM from the terminal

We built a CRM MCP server that lets Claude Code interact with our Firestore-backed CRM:

```
/crm_list_contacts → Browse contacts with search/filter
/crm_contact_360   → Full customer view: deals, tickets, timeline
/crm_draft_followup → Generate follow-up email from context
/crm_deal_score    → Score deal probability
```

**Why this matters**: The CEO is the sales team. Context-switching between terminal, CRM dashboard, and email kills flow. With MCP, you check a lead's status, draft a follow-up, and get back to code — without leaving Claude Code.

## Regional prospecting with Claude

We use Claude Code to build and maintain prospecting backlogs per region:

1. **Define ICP** (Ideal Customer Profile) per region — Claude helps research local industry composition
2. **Generate prospect lists** from public data (Chamber of Commerce, LinkedIn, industry directories)
3. **Score and prioritize** based on ICP match, company size, digital presence
4. **Draft personalized outreach** in the local language (Portuguese for Aveiro/Porto, Spanish for Madrid)

Each region gets its own backlog file with prospects, status, and next actions.

## Content generation pipeline

### Editorial calendar (Q2 2026)
- 4 content pillars rotating weekly: Research, Client Cases, Methodology, Culture
- 5 channels: LinkedIn, Facebook, Instagram, TikTok, YouTube Shorts
- 1 post/week/channel = 60 posts/quarter
- Claude generates the content calendar and first drafts
- Radaar.io handles scheduling and analytics

### The workflow

```
1. Claude generates monthly content calendar (4 pillars × 5 channels)
2. Human reviews and adjusts topics
3. Claude drafts each post adapted to channel format:
   - LinkedIn: 200-400 words, professional tone
   - Facebook: 80-150 words + image prompt
   - Instagram: 50-100 words carousel
   - TikTok/YouTube: 60-second script
4. Human reviews, uploads to Radaar.io
5. Radaar publishes on schedule
```

## Google Ads with Ad Grants

We manage Google Ads for nonprofit clients under the Ad Grants program. Claude Code generates campaigns following strict Ad Grants rules:

- Max $329/day
- Bidding: Maximize Conversions only
- Min 2 ad groups per campaign
- Quality Score >= 3 (or auto-pause)
- CTR >= 5% (or lose the grant)
- CSV exports: UTF-8 BOM for web, UTF-16LE for Google Ads Editor

The BI MCP server provides Google Ads performance data directly in Claude Code.

## Support tickets from the terminal

The ticketing MCP connects to soporte.proportione.com:

```
/ticketing_stats         → Dashboard: open, resolved, SLA breaches
/ticketing_list_tickets  → Filter by status, search by email/subject
/ticketing_get_ticket    → Full ticket with message timeline
/ticketing_reply         → Add staff reply or internal note
```

This means a support request can be triaged, investigated (by reading code), and answered — all in one Claude Code session.

## Lessons learned

1. **MCP servers are the killer feature for non-engineering workflows**. Being able to check CRM, tickets, and analytics without leaving the terminal eliminates context-switching.

2. **Lifetime deals + Claude Code > expensive SaaS subscriptions**. Most marketing SaaS does 3 things: store data, render UI, and process rules. Claude Code can generate rules and process data; the SaaS just needs to store and display.

3. **The "Capataz y Obrero" model extends to marketing**. Claude (capataz) generates the strategy and content calendar. The local LLM (obrero) can handle routine content variations and translations.

4. **One person with Claude Code can maintain a presence on 5 social channels**. The bottleneck isn't content creation — it's review quality. Allocate human time to reviewing, not writing.
