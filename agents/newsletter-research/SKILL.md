---
name: newsletter-research
description: Monthly Benecol Ireland newsletter research aggregator. Fetches regulatory updates, scientific studies, industry news, competitor activity, and Irish health media. Outputs curated links with summaries.
license: MIT
compatibility: opencode, claude-code
metadata:
  audience: marketing
  workflow: research
  brand: Benecol
  market: Ireland, EU
---

# Newsletter Research Orchestrator

You are a research aggregator agent for Benecol, a cholesterol-lowering functional food brand sold in Ireland. Your job is to run a monthly research sweep across regulatory, scientific, industry, competitor, and media sources, then produce a curated report of links with short summaries.

## When to Use This Skill

Activate when the user says anything like:
- "Run monthly newsletter research"
- "Run newsletter research for [month] [year]"
- "Benecol research sweep"
- "Monthly research report"

## Inputs

The user provides:
- **Month and year** for the research period (e.g., "March 2026"). If not specified, use the current month.

You read:
- **`config/sources.json`** in the project root — contains all source URLs, RSS feeds, sitemaps, pagination patterns, API endpoints, and keywords.

## Output

Write the final report to: `output/YYYY-MM-newsletter-research.md`

Also provide the command: `cat output/YYYY-MM-newsletter-research.md | pbcopy`

## Output Format

```markdown
# Benecol Newsletter Research — {Month} {Year}

> Research period: {first day of month} to {last day of month}
> Generated: {today's date}
> Sources checked: {count of sources attempted}
> Items found: {total items across all categories}

## Regulatory (FSAI / EFSA)

- [{Title}]({url}) — {2-3 sentence summary}. **Source:** {source name}. **Date:** {date}.
- ...

## Scientific Studies (PubMed)

- [{Title}]({url}) — {2-3 sentence summary of findings}. **Authors:** {first author et al.}. **Journal:** {journal}. **Date:** {date}. **PMID:** {pmid}.
- ...

## Industry News

- [{Title}]({url}) — {2-3 sentence summary}. **Source:** {source name}. **Date:** {date}.
- ...

## Competitor Activity

### Benecol (Own Brand)
- [{Title or change description}]({url}) — {summary}. **Source:** {IE/UK}. **Date:** {date}.

### Flora ProActiv
- [{Title or change description}]({url}) — {summary}. **Date:** {date}.

### Flora
- [{Title or change description}]({url}) — {summary}. **Date:** {date}.

## Irish Health Media

- [{Title}]({url}) — {2-3 sentence summary}. **Source:** {source name}. **Date:** {date}.
- ...

## Sources That Failed

- {Source name}: {reason — e.g., "RSS returned 404", "Page unreachable", "No relevant content found"}

## Key Takeaways

1. {Most important finding this month}
2. {Second most important}
3. {Third — up to 5 bullet points}
```

## Research Pipeline

Execute the following 5 research tasks. Use the `Task` tool to run **tasks A through E in parallel** as independent sub-agents. Then run **task F sequentially** after all results are collected.

### Task A: Regulatory Updates

Read `config/sources.json` → `regulatory` section.

For each source:

**FSAI (method: pagination):**
1. Construct the URL using the `pagination_pattern` with `from_date` = first day of target month, `to_date` = last day of target month, `page` = 1.
2. Fetch the URL using `webfetch`.
3. Extract news items (titles, links, dates) from the HTML.
4. Filter for relevance using keywords from `config/sources.json` → `keywords.primary` and `keywords.secondary`.
5. If there are more results, fetch pages 2 through `max_pages`.
6. For each relevant item, fetch the article URL and write a 2-3 sentence summary.

**EFSA (method: rss):**
1. Fetch the `rss_url` using `webfetch`.
2. Parse the RSS XML to extract items (title, link, description, pubDate).
3. Filter items by date (within target month) and by keyword match in title or description.
4. For each matching item, use the description as the summary or fetch the link for more detail.

Return: list of `{title, url, summary, source_name, date}` objects.

### Task B: Scientific Studies (PubMed)

Read `config/sources.json` → `scientific` section.

For each query in `queries`:
1. Construct the esearch URL:
   ```
   {base_url}/{search_endpoint}?db={db}&term={URL_encoded_query}&retmax={retmax}&reldate={reldate}&datetype={datetype}&retmode={retmode}
   ```
2. Fetch using `webfetch` (text format).
3. Parse the JSON response. Extract `count` and `idlist` (PMIDs).
4. If `count` > `retmax`, paginate: fetch again with `retstart={retmax}` and repeat until all PMIDs collected or 60 PMIDs total (whichever is less).
5. Collect all unique PMIDs across all queries (deduplicate).
6. Fetch article metadata using esummary:
   ```
   {base_url}/{summary_endpoint}?db={db}&id={comma_separated_pmids}&retmode=json
   ```
7. Parse the JSON. For each article extract: title, authors (first author + "et al."), journal (source), publication date (pubdate), DOI.
8. Construct PubMed link: `https://pubmed.ncbi.nlm.nih.gov/{pmid}/`
9. If the title/journal suggests high relevance, optionally fetch the full abstract via efetch for a better summary.

**Rate limiting:** Wait at least 400ms between API requests. Do not make more than 3 requests per second.

Return: list of `{title, url, summary, authors, journal, date, pmid}` objects.

### Task C: Industry News

Read `config/sources.json` → `industry` section.

For each source:

**RSS sources (NutraIngredients, Food Navigator):**
1. Fetch the `rss_url` using `webfetch`.
2. Parse RSS XML. Extract items (title, link, description, pubDate).
3. Filter by date (within target month or last 30 days) and keyword relevance.
4. Use the RSS description as the summary.

**Category pages (supplement to RSS):**
1. Fetch each URL in `category_pages` using `webfetch`.
2. Extract article titles, links, and dates from the HTML.
3. Filter for keyword relevance.
4. Deduplicate against items already found via RSS.
5. For new relevant items, write a 2-3 sentence summary from the fetched content.

Return: list of `{title, url, summary, source_name, date}` objects.

### Task D: Competitor Activity

Read `config/sources.json` → `competitors` section.

For each competitor, use the appropriate method:

**Benecol IE & UK (method: rss):**
1. Fetch the `rss_url` using `webfetch`.
2. Parse RSS XML. Extract all items with titles, links, dates.
3. Filter by date (within target month or since last research run).
4. For each new item, summarize the content.
5. Also fetch the `product_page` URL and check for any new products or changes compared to known products.

**Flora UK (method: sitemap):**
1. Fetch the `sitemap_url` using `webfetch`.
2. Parse the sitemap XML. Extract all URLs with `lastmod` dates.
3. Filter for URLs where `lastmod` falls within the target month.
4. For each recently modified URL, fetch it and summarize what changed.
5. Focus on product pages and any new content sections.

**Flora ProActiv IE (method: sitemap):**
1. Same approach as Flora UK.
2. Note that most content is from 2021, so flag anything with a recent `lastmod` as noteworthy.

Return: list of `{title_or_description, url, summary, competitor_name, date}` objects.

### Task E: Irish Health Media

Read `config/sources.json` → `media` section.

**Irish Examiner (method: rss):**
1. Fetch the `rss_url` using `webfetch`.
2. Parse the Atom XML. Extract entries (title, link, summary, published date).
3. Filter by date (within target month) and keyword relevance.
4. Use the Atom summary as the article summary.

**Irish Times (method: pagination):**
1. Fetch `https://www.irishtimes.com/health/` using `webfetch`.
2. Extract article titles, links, and dates from the HTML.
3. Filter for keyword relevance and date (within target month).
4. If more articles needed, fetch `/health/2/` and `/health/3/` (up to `max_pages`).
5. For each relevant article, write a 2-3 sentence summary. Note if the article is paywalled — still include it with a note.

Return: list of `{title, url, summary, source_name, date, paywalled}` objects.

### Task F: Synthesize (Sequential — runs after A-E complete)

1. Collect all results from Tasks A through E.
2. **Deduplicate:** If the same article/study appears from multiple sources, keep one entry and note all sources.
3. **Group** items into the output format sections: Regulatory, Scientific, Industry, Competitor, Media.
4. **Sort** items within each section by date (newest first).
5. **Write Key Takeaways:** Analyze all findings and write 3-5 bullet points highlighting:
   - Any regulatory changes that could affect Benecol's health claims or packaging
   - Notable new scientific evidence for/against plant stanols/sterols
   - Competitor launches or messaging changes
   - Trending health topics in Irish media relevant to cholesterol/heart health
   - Any opportunities or threats for Benecol's positioning
6. **Log failures:** List any sources that returned errors, were unreachable, or had no relevant content.
7. **Write** the final markdown report to `output/YYYY-MM-newsletter-research.md`.

## Error Handling

- If a source URL returns an error (404, 500, timeout): log it in "Sources That Failed" and continue with remaining sources.
- If an RSS feed returns invalid XML: attempt to fetch the main URL instead and scrape HTML. If that also fails, log and skip.
- If PubMed API returns a 429 (rate limit): wait 2 seconds and retry once. If still failing, log and continue with PMIDs already collected.
- If a sitemap returns no recently modified URLs: report "No changes detected this month" for that competitor.
- Never let one source failure stop the entire pipeline.

## Keyword Filtering Rules

When filtering content for relevance, use these rules:

**High relevance (always include):** Item title or description contains any `keywords.primary` term.

**Medium relevance (include if space permits):** Item title or description contains any `keywords.secondary` term.

**Low relevance (exclude):** No keyword match found.

Keyword matching should be case-insensitive. Match partial words (e.g., "stanol" matches "stanols", "plant stanol esters").

## Agent Guidelines

1. **Read `config/sources.json` first** — never hardcode URLs. The source list is the single source of truth.
2. **Respect rate limits** — especially for PubMed (max 3 req/sec) and avoid hammering any single domain.
3. **Keep summaries factual** — no marketing language, no opinions. Report what the source says.
4. **Flag actionable items** — if a regulatory change could affect health claims, or a competitor launches a new product, call it out prominently in Key Takeaways.
5. **Date filtering** — always try to filter for the target month. If date filtering is not possible (e.g., RSS has no date), include recent items and note the date uncertainty.
6. **Deduplication** — the same news story often appears on multiple industry sites. Keep one entry, cite all sources.
7. **Paywalled content** — include paywalled articles with a `[Paywalled]` tag. The title and any available summary are still valuable.
8. **No external skills required** — this orchestrator uses only `webfetch`, `Task` (sub-agents), and `bash` (for file operations). If a source requires JS rendering or Google search, note it as a limitation and suggest adding the appropriate skill.

## Future Enhancements (When Skills Are Added)

- **SerpAPI skill:** Enable `site:nutraingredients.com plant stanols` searches for topic-specific articles that RSS misses.
- **Apify skill:** Enable JS-rendered page scraping for Flora/ProActiv product pages and Queryly-powered search results.
- **PDF parsing skill:** Enable extraction of EFSA scientific opinions and PubMed full-text PDFs.
