# Dataset Diff & Change Detector (Remote MCP Server)

[![Dataset Diff & Change Detector (Remote MCP Server) MCP connector](https://glama.ai/mcp/connectors/io.github.Nero-Engine/dataset-diff-detector/badges/score.svg)](https://glama.ai/mcp/connectors/io.github.Nero-Engine/dataset-diff-detector)

**Compare two versions of the same list of JSON rows and get exactly what changed.** Hand it yesterday's price list and today's, last week's product feed and this week's, or two exports of the same CRM, name the field that identifies a row, and it tells you which rows were **added**, **removed** or **changed**, down to which fields moved on each one.

Built for AI agents. No install, no API key, no signup. Connect by URL and call it.

```
https://dataset-diff-detector.nerolabs.workers.dev/mcp
```

Free to use while in early access.

## What it does

One call compares a before snapshot (`oldRows`) with an after snapshot (`newRows`):

1. **Matches** rows across the two sides on `keyFields`, such as `sku`, `id` or `email` (one field or several). Without a key it matches on full row content, so an edited row shows as one removed row plus one added row.
2. **Compares** every field on each matched pair, minus any `ignoreFields` (timestamps that always differ), or only the `compareFields` you name.
3. **Returns** one row per difference with its `status`, `key`, `oldValues`, `newValues` and `changedFields`, plus a summary counting added, removed, changed and unchanged rows. Turn on `includeUnchanged` to get the unchanged rows back too.

Comparison is exact, so nothing is quietly glossed over: `"24.99"` as text and `24.99` as a number count as a change, as do `"Blue"` and `"blue "`, and `null` becoming a value. Nested objects compare by value, whatever order their keys are in.

It is honest about messy input. The summary warns when a key value appears twice on one side (the last row is kept and the count is reported), when rows have no key field at all, and when a field you named appears in no row, which is usually a typo.

## Tools

| Tool | What it does |
|---|---|
| `list_capabilities` | Lists the exact matching and comparison rules, the output shape and the row limit. Processes no data. |
| `diff_rows` | Compares `oldRows` with `newRows` and returns the differences plus a summary. |

## Connect

**Claude Code**

```bash
claude mcp add --transport http dataset-diff-detector https://dataset-diff-detector.nerolabs.workers.dev/mcp
```

**Claude Desktop / claude.ai:** Settings, Connectors, Add custom connector, paste the URL above.

**Cursor, Windsurf, VS Code and other MCP clients**

```json
{
  "mcpServers": {
    "dataset-diff-detector": {
      "url": "https://dataset-diff-detector.nerolabs.workers.dev/mcp"
    }
  }
}
```

## Example

Yesterday's and today's price list go in, with the scrape timestamp ignored:

```json
{
  "oldRows": [
    {"sku": "A100", "title": "Blue Widget", "price": 19.99, "stock": 42, "scrapedAt": "2026-09-11T08:00:00Z"},
    {"sku": "A101", "title": "Red Widget", "price": "24.99", "stock": 0, "scrapedAt": "2026-09-11T08:00:00Z"},
    {"sku": "A102", "title": "Green Widget", "price": 15.5, "stock": 8, "scrapedAt": "2026-09-11T08:00:00Z"},
    {"sku": "A104", "title": "Purple Widget", "price": 9.99, "discount": null, "scrapedAt": "2026-09-11T08:00:00Z"},
    {"sku": "A105", "title": "Orange Widget", "price": 30, "stock": 3, "scrapedAt": "2026-09-11T08:00:00Z"}
  ],
  "newRows": [
    {"sku": "A100", "title": "Blue Widget", "price": 17.99, "stock": 30, "scrapedAt": "2026-09-12T08:00:00Z"},
    {"sku": "A101", "title": "Red Widget", "price": 24.99, "stock": 0, "scrapedAt": "2026-09-12T08:00:00Z"},
    {"sku": "A102", "title": "Green Widget", "price": 15.5, "stock": 8, "scrapedAt": "2026-09-12T08:00:00Z"},
    {"sku": "A104", "title": "Purple Widget", "price": 9.99, "discount": 0.1, "scrapedAt": "2026-09-12T08:00:00Z"},
    {"sku": "A106", "title": "Yellow Widget", "price": 12, "stock": 100, "scrapedAt": "2026-09-12T08:00:00Z"}
  ],
  "keyFields": ["sku"],
  "ignoreFields": ["scrapedAt"]
}
```

Five differences come out. A105 was removed, A106 was added, A100 changed price and stock, A101's price turned from text into a number, and A104 gained a discount. A102 matched exactly and is counted as unchanged:

```json
{
  "rows": [
    {"status":"removed","key":{"sku":"A105"},"oldValues":{"sku":"A105","title":"Orange Widget","price":30,"stock":3,"scrapedAt":"2026-09-11T08:00:00Z"}},
    {"status":"changed","key":{"sku":"A100"},"oldValues":{"sku":"A100","title":"Blue Widget","price":19.99,"stock":42,"scrapedAt":"2026-09-11T08:00:00Z"},"newValues":{"sku":"A100","title":"Blue Widget","price":17.99,"stock":30,"scrapedAt":"2026-09-12T08:00:00Z"},"changedFields":["price","stock"]},
    {"status":"changed","key":{"sku":"A101"},"oldValues":{"sku":"A101","title":"Red Widget","price":"24.99","stock":0,"scrapedAt":"2026-09-11T08:00:00Z"},"newValues":{"sku":"A101","title":"Red Widget","price":24.99,"stock":0,"scrapedAt":"2026-09-12T08:00:00Z"},"changedFields":["price"]},
    {"status":"changed","key":{"sku":"A104"},"oldValues":{"sku":"A104","title":"Purple Widget","price":9.99,"discount":null,"scrapedAt":"2026-09-11T08:00:00Z"},"newValues":{"sku":"A104","title":"Purple Widget","price":9.99,"discount":0.1,"scrapedAt":"2026-09-12T08:00:00Z"},"changedFields":["discount"]},
    {"status":"added","key":{"sku":"A106"},"newValues":{"sku":"A106","title":"Yellow Widget","price":12,"stock":100,"scrapedAt":"2026-09-12T08:00:00Z"}}
  ],
  "summary": {
    "oldRecordCount": 5,
    "newRecordCount": 5,
    "counts": {"added": 1, "removed": 1, "changed": 3, "unchanged": 1},
    "differenceCount": 5,
    "returnedRowCount": 5,
    "unchangedRowsIncluded": false,
    "keyFieldsUsed": ["sku"],
    "ignoreFields": ["scrapedAt"],
    "duplicateRowsSkipped": {"old": 0, "new": 0},
    "rowsMissingKeyFields": {"old": 0, "new": 0},
    "warnings": []
  }
}
```

Removed rows come first, then changed rows in the order of `oldRows`, then added rows in the order of `newRows`.

## Skills

This server also publishes ready-made playbooks (Skills over MCP), so Claude and other agents know the steps for the whole job, not just each tool. Each is a `SKILL.md` at `skill://<name>/SKILL.md`, listed by `skills/list` and `resources/list`, read with `resources/read`.

- `what-changed-between-snapshots`: Compare two snapshots of the same list (yesterday's and today's prices, product feed, job board, CRM export, competitor catalogue) and report what was added, removed and changed, field by field.

## Limits

Up to **500 rows per call**, `oldRows` and `newRows` combined. For bigger lists, split them across several calls and keep the same key range on both sides of each call (for example SKUs A to M in one call, N to Z in the next), otherwise a row in one call looks removed while its match in another looks added. Anything larger returns a clear message rather than failing silently.

## Privacy

Your rows are processed in memory and never stored. To see which tools get used, each call records the tool name, row counts, whether it succeeded, the client name your app reports, the country and a one-way hashed caller ID. Your data, your arguments and your IP address are never kept in that log.

## Also available

The same engine runs on the Apify Store as [Dataset Diff & Change Detector](https://apify.com/nerolabs/dataset-diff-detector), which adds "since last run" snapshot mode (name a comparison and every run reports only what changed since the previous one, so you never supply the old side again), reads Apify datasets, CSV, Excel and JSON files and Google Sheets on either side, compares up to 100,000 rows a run, exports a CSV or Excel diff report, keeps a running change log in a named dataset, and posts each result to a webhook.

Built by **Nero Labs**.

## Gemini CLI

- **Gemini CLI:** `gemini extensions install https://github.com/Nero-Engine/dataset-diff-detector-mcp`
