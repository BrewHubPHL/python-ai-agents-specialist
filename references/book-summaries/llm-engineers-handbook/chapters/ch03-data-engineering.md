# Chapter 3: Data Engineering

## Core Idea
Build a ZenML-orchestrated ETL pipeline that crawls personal content from Medium, Substack, GitHub, and LinkedIn, normalizes it into three document categories (article, repository, post), and loads raw documents into MongoDB — decoupled from the downstream RAG feature pipeline.

## Frameworks Introduced
- **ETL for LLM data**: Extract (crawl) → Transform (clean/normalize) → Load (MongoDB warehouse).
  - When to use: Real-world LLM projects without a static Kaggle dataset.
  - How: Input user + link list → dispatcher picks crawler → standardized documents per author.
- **Crawler dispatcher pattern**: Route URLs to specialized crawlers by domain.
  - When to use: Multiple content sources with different access patterns.
  - How: `urlparse` domain → `MediumCrawler`, `GitHubCrawler`, `CustomArticleCrawler`, or `LinkedInCrawler`.
- **Data category abstraction**: Collapse sources into article / repository / post — not per-platform types.
  - When to use: Adding new sources without rewriting downstream pipelines.
  - How: New crawler outputs existing category; source URL lives in metadata.

## Key Concepts
- **ODM (Object Document Mapper)**: MongoDB document classes with `get_or_create` patterns.
- **GitHubCrawler**: Clone repo → parse file tree → normalize code files.
- **MediumCrawler**: Login → extract article HTML → clean text.
- **CustomArticleCrawler**: Generic HTML scrape for Substack/blogs (no login).
- **LinkedInCrawler**: Login → crawl user feed posts.
- **digital_data_etl pipeline**: `get_or_create_user` → `crawl_links` ZenML steps.
- **Warehouse ↔ feature pipeline contract**: ETL writes MongoDB; feature pipeline reads independently on different schedules.

## Mental Models
- Categorize by *document shape* (article/repo/post), not by *platform* — source is metadata, not schema.
- Hundreds of docs suffice for a PoC; thousands needed for production-quality fine-tuning — architect for easy source expansion.
- MongoDB as "warehouse" is a pragmatic PoC choice for unstructured text at small scale; migrate to Snowflake/BigQuery at millions of docs.

## Anti-patterns
- **Per-source document classes**: Adding X/Twitter would require changing feature pipeline, training, and inference — use category abstraction instead.
- **Coupling ETL to embedding**: Data collection and vector ingestion must stay separate pipelines communicating only through the warehouse.
- **Transactional DB at big-data scale**: MongoDB works for hundreds of docs; not for millions without a real warehouse.

## Code Examples
```python
@pipeline
def digital_data_etl(user_full_name: str, links: list[str]) -> str:
    user = get_or_create_user(user_full_name)
    last_step = crawl_links(user=user, links=links)
    return last_step.invocation_id
```
- **What it demonstrates**: ZenML pipeline entry point for data collection.

## Worked Example
**Pipeline signature**:
- Input: author full name + list of profile/article/repo URLs
- Output: raw documents in MongoDB under that user
- Crawlers: Medium (article), CustomArticle (Substack/blogs), GitHub (repository), LinkedIn (posts)
- Dispatcher: domain detection selects crawler; unknown domains fall back to CustomArticleCrawler

## Key Takeaways
1. Real LLM products start with autonomous, schedulable data collection — not static CSVs.
2. Three document categories (article, repository, post) keep the architecture extensible.
3. ETL and feature pipelines communicate only through MongoDB — independent schedules and scaling.
4. ZenML steps attach metadata artifacts (user_id, query params) for reproducibility.
5. Selenium/login crawlers need troubleshooting paths; backup data import is a valid fallback.

## Connects To
- **Ch 1**: Data collection pipeline from FTI architecture design.
- **Ch 4**: Feature pipeline reads MongoDB → chunks/embeds → Qdrant.
- **Ch 5**: Scraped data becomes instruction dataset for SFT.
