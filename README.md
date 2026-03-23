# LLM Inference Pricing Scraper — v2.0.0

A chatbot that scrapes LLM Inference Serving websites to research costs of serving various LLMs. Built with an MCP Server connected to Firecrawl's API, storing data in a SQLite database.

## Websites scraped

- **cloudrift**: https://www.cloudrift.ai/inference
- **deepinfra**: https://deepinfra.com/pricing
- **fireworks**: https://fireworks.ai/pricing#serverless-pricing
- **groq**: https://groq.com/pricing

## Setup

1. Create and activate a virtual environment:
   ```powershell
   uv venv
   & .venv\Scripts\Activate.ps1
   ```
2. Install dependencies:
   ```powershell
   uv sync
   ```
3. Create a `key.env` file with your API keys:
   ```
   ANTHROPIC_API_KEY=your_key
   FIRECRAWL_API_KEY=your_key
   ```
4. Run the chatbot:
   ```powershell
   python starter_client.py
   ```

## Usage

At the `Query:` prompt, use the following commands:

| Command | Description |
|---------|-------------|
| `scrape these sites: {...}` | Scrapes websites and stores content |
| `Compare X and Y's costs for model Z` | Compares pricing using stored data |
| `show data` | Displays recently stored pricing plans from SQLite |
| `quit` / `exit` | Exits the chatbot |

### Example prompts

```
scrape these sites: {'cloudrift': 'https://www.cloudrift.ai/inference', 'deepinfra': 'https://deepinfra.com/pricing', 'fireworks': 'https://fireworks.ai/pricing#serverless-pricing', 'groq': 'https://groq.com/pricing'}
```
```
Compare cloudrift ai and deepinfra's costs for deepseek v3
```
```
show data
```

## Architecture

- **`starter_server.py`** — MCP server with two tools:
  - `scrape_websites`: scrapes URLs via Firecrawl and persists content + metadata
  - `extract_scraped_info`: retrieves stored content by provider name, URL or domain
- **`starter_client.py`** — MCP client that:
  - Connects to 3 MCP servers: custom scraper, SQLite, filesystem
  - Drives tool use via Claude LLM
  - Automatically extracts and stores structured pricing data in SQLite
  - Answers follow-up queries from stored data (no re-scraping)
- **`server_config.json`** — MCP server configuration

## Changelog

### v2.0.0
- Fixed SQL injection vulnerability in pricing data insertion (escaped single quotes)
- Added null-check for server lookup before tool execution
- Scrape commands now bypass database lookup and always trigger fresh scraping
- Added automatic `extract_scraped_info` calls after scraping (client-driven, not LLM-driven)
- Improved output visibility: shows sites being scraped, chars extracted per provider, DB storage confirmation
- Fixed JSON parsing error when LLM returns multiple concatenated JSON objects
- Extraction prompt now requests a single root array covering all providers
- Suppressed verbose raw extraction logs for cleaner terminal output
- `_query_from_database` now logs when answering from SQLite vs re-scraping
- Connection startup now lists all 3 MCP servers with their names

