# Benecol Newsletter Research Tool

A monthly research assistant that automatically gathers news, studies, and competitor updates relevant to Benecol's cholesterol-lowering products in Ireland. It saves you hours of manual Googling and website checking each month.

## What Does This Tool Do?

Every month, you run a single command and the tool goes out and checks **10+ sources** for you:

| What it checks | Where it looks |
|---|---|
| **Regulatory updates** | FSAI (Food Safety Authority of Ireland), EFSA (EU food safety) |
| **New scientific studies** | PubMed (medical research database) |
| **Industry news** | NutraIngredients, Food Navigator Europe |
| **Competitor activity** | Benecol IE & UK sites, Flora UK, Flora ProActiv Ireland |
| **Irish health media** | Irish Times Health, Irish Examiner Health |

It then produces a **single report** with links and short summaries, grouped by category — ready for you to use when writing the newsletter.

## What You Get

A markdown file in the `output/` folder (e.g. `output/2026-03-newsletter-research.md`) containing:

- **Regulatory** — Any new rules or updates about health claims, functional foods, or plant sterols
- **Scientific Studies** — Recent research papers about plant stanols, cholesterol reduction, etc.
- **Industry News** — What's happening in the supplement and functional food world
- **Competitor Activity** — New blog posts, product launches, or website changes from Benecol (own brand tracking) and Flora ProActiv
- **Irish Health Media** — Cholesterol and heart health articles in Irish newspapers
- **Key Takeaways** — A short summary of the 3-5 most important things you should know this month

Each item includes a **clickable link** and a **2-3 sentence summary** so you can quickly scan what matters.

## How to Run It

### First-Time Setup

You need one of these AI coding tools installed:

- [opencode](https://opencode.ai) — open-source CLI tool
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — Anthropic's CLI tool

### Running the Monthly Research

1. Open your terminal
2. Navigate to this project folder:
   ```
   cd ~/src/marketing-report-generation
   ```
3. Start opencode (or Claude Code):
   ```
   opencode
   ```
4. Type this command:
   ```
   @agents/newsletter-research/SKILL.md Run monthly newsletter research for March 2026
   ```
   (Replace "March 2026" with whatever month you need)

5. Wait 3-5 minutes while it fetches all sources
6. Your report will be at `output/2026-03-newsletter-research.md`

## How It Works — The Agentic Flow

This tool uses an **agentic workflow** — instead of a single script running step by step, it works more like a team of assistants collaborating on a task.

### What Is an Agentic Workflow?

Think of it like delegating research to a team. You give one instruction ("research this month's newsletter topics"), and an **orchestrator** breaks that down into smaller tasks and hands them out to **specialist agents** who work in parallel — just like assigning different team members to cover different beats.

```
You: "Run newsletter research for March 2026"
 │
 ▼
Orchestrator (reads your source list and kicks off the team)
 │
 ├── Agent A: Checks FSAI and EFSA for regulatory news
 ├── Agent B: Searches PubMed for new scientific studies
 ├── Agent C: Reads NutraIngredients and Food Navigator RSS feeds
 ├── Agent D: Checks Benecol, Flora, and ProActiv for competitor changes
 └── Agent E: Scans Irish Times and Irish Examiner health sections
      │
      ▼  (all 5 work at the same time)
      │
 Orchestrator collects all results
      │
      ▼
 Final step: Removes duplicates, organises by category, writes the report
```

### Why This Matters for Your Work

Agentic workflows aren't limited to newsletter research. The same pattern applies to any repetitive task where you:

1. **Gather information from multiple places** (websites, databases, feeds)
2. **Filter for what's relevant** (keywords, dates, topics)
3. **Summarise and organise** the results into something useful

Examples of other things you could build with this same approach:

- **Social media content calendar** — an agent that scans trending health topics and suggests post ideas
- **Competitor social monitoring** — agents that check competitor social accounts and summarise their recent activity
- **Campaign performance reports** — agents that pull data from multiple platforms and compile a single brief
- **Event/conference tracking** — agents that monitor industry event sites and flag relevant upcoming events

Each of these would follow the same pattern: one orchestrator skill file (like our `SKILL.md`), one config file with your sources, and an output folder for results.

### Building Your Own Agentic Workflows

If you want to create a new workflow for a different task:

1. **Define what you want to monitor** — list the websites, feeds, or data sources
2. **Write the instructions** — create a `SKILL.md` file that tells the AI what to do, step by step
3. **Configure your sources** — put URLs and keywords in a `sources.json` file
4. **Run it** — point the AI tool at your skill file and give it a prompt

The AI handles the rest — fetching, reading, filtering, summarising, and compiling. You just review the output.

## Project Structure

```
marketing-report-generation/
├── agents/newsletter-research/   ← The AI instructions (you don't need to edit this)
│   └── SKILL.md
├── config/
│   └── sources.json              ← The list of websites and keywords to monitor
└── output/                       ← Monthly reports appear here
```

## Updating the Source List

Want to add a new website to monitor, or change the keywords? Edit `config/sources.json`.

### Adding a New Website with an RSS Feed

Open `config/sources.json` and add an entry to the relevant section. For example, to add a new industry news source:

```json
{
  "name": "New Source Name",
  "url": "https://www.example.com/",
  "rss_url": "https://www.example.com/feed/",
  "method": "rss",
  "notes": "Description of what to look for here"
}
```

### Changing Keywords

In the same file, scroll to the `keywords` section at the bottom. Add or remove terms:

```json
"keywords": {
  "primary": ["plant stanols", "plant sterols", "cholesterol lowering", "Benecol"],
  "secondary": ["functional foods", "heart health", "LDL cholesterol"]
}
```

**Primary keywords** = always included in results. **Secondary keywords** = included when relevant.

## Limitations

- **Some websites can't be fully read** — Sites that rely heavily on JavaScript (like Flora and Flora ProActiv) are monitored via their sitemaps rather than full page content. The tool flags when this happens.
- **Paywalled articles** — Irish Times articles behind a paywall are still listed with their title and any available summary, marked as `[Paywalled]`.
- **Search doesn't work everywhere** — Some sites (NutraIngredients, Food Navigator) have search features that require a browser. The tool uses RSS feeds and category pages instead.
- **Not real-time** — This is designed for monthly research sweeps, not live monitoring.

## Future Improvements

These can be added when needed (they require extra setup):

- **Google search integration** — Search industry sites for specific topics when RSS doesn't cover them
- **Advanced web scraping** — Better access to JavaScript-heavy competitor sites
- **PDF reading** — Extract information from EFSA scientific opinion documents and full research papers
