# @paperjsx/mcp-server

Local MCP server that gives AI agents a dozen document-generation tools — PPTX presentations, DOCX reports and contracts, PDF invoices and charts, XLSX spreadsheets — by wrapping the free-tier `@paperjsx/json-to-*` packages. Runs in-process, no API key, no network calls.

[![npm](https://img.shields.io/npm/v/@paperjsx/mcp-server)](https://www.npmjs.com/package/@paperjsx/mcp-server)
[![license](https://img.shields.io/npm/l/@paperjsx/mcp-server)](./LICENSE)

```bash
npx -y @paperjsx/mcp-server
```

Requires Node.js `>=18`.

## What It Does

Registers MCP tools so any MCP-capable client (Claude Desktop, Cursor, VS Code Copilot, Windsurf, Cline, Gemini CLI, Claude.ai custom connectors) can ask an agent to *actually produce* a document from a JSON spec.

Everything runs locally via:

- `@paperjsx/json-to-pptx` — PPTX presentations
- `@paperjsx/json-to-docx` — DOCX reports, invoices, contracts
- `@paperjsx/json-to-xlsx` — XLSX spreadsheets with native charts
- `@paperjsx/json-to-pdf` — PDF invoices, reports, chart documents

There is no cloud backend, no account, no API key to set. The server writes generated files to disk and returns the absolute path so the client can attach it or surface it in-line.

## Tool Catalog

| Tool                          | Format | Use case                                             |
| ----------------------------- | ------ | ---------------------------------------------------- |
| `generate_presentation`       | PPTX   | Decks from a JSON slide spec (title / content / chart / comparison / stats) |
| `generate_invoice`            | PDF    | Invoices with sender, recipient, line-items, totals  |
| `generate_report`             | PDF    | Markdown-driven PDF reports with cover page + TOC    |
| `generate_chart_document`     | PDF    | Multi-chart documents with insights + optional data table |
| `generate_report_docx`        | DOCX   | Word reports with sections, headers, page numbers    |
| `generate_contract_docx`      | DOCX   | Structured contracts with clause numbering           |
| `generate_invoice_docx`       | DOCX   | Word invoices                                        |
| `generate_spreadsheet`        | XLSX   | Multi-sheet workbooks, formulas, native charts       |
| `validate_spreadsheet`        | —      | Validate an `.xlsx` against the Zod schema           |
| `repair_spreadsheet`          | XLSX   | Repair a corrupted workbook                          |
| `list_templates`              | —      | Discover reusable JSON templates (name / formats / sample data) |
| `list_components`             | —      | Document layout primitives shared across formats     |
| `batch_generate`              | —      | Run several generation jobs in one call              |
| `remediate_accessibility`     | —      | Apply conservative accessibility fixes to a JSON spec |

Every tool exposes a JSON Schema (derived from its Zod definition) so the agent sees accurate parameter hints and example inputs.

## Configuration

### Environment Variables

| Variable                       | Required | Default      | Notes                               |
| ------------------------------ | -------- | ------------ | ----------------------------------- |
| `PAPERJSX_OUTPUT_DIR`          | No       | OS temp dir  | Where generated files are written   |
| `PAPERJSX_ARTIFACT_TTL_HOURS`  | No       | `24`         | Auto-cleanup horizon                |
| `PAPERJSX_MCP_METRICS_ENDPOINT`| No       | —            | Optional telemetry sink URL         |
| `PAPERJSX_MCP_METRICS_TOKEN`   | No       | —            | Optional bearer token for the sink  |

No authentication is required to use any tool.

### Claude Desktop

`claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "paperjsx": {
      "command": "npx",
      "args": ["-y", "@paperjsx/mcp-server"]
    }
  }
}
```

### Cursor

`.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "paperjsx": {
      "command": "npx",
      "args": ["-y", "@paperjsx/mcp-server"]
    }
  }
}
```

### VS Code Copilot

`.vscode/mcp.json`:

```json
{
  "servers": {
    "paperjsx": {
      "command": "npx",
      "args": ["-y", "@paperjsx/mcp-server"]
    }
  }
}
```

### Windsurf / Cline / Gemini CLI

Use the same `mcpServers` block as Claude Desktop — the Windsurf, Cline, and Gemini config files all accept it.

## Example Agent Prompts

Once configured, just ask:

> "Generate an invoice for Acme Corp: 1 × Enterprise Workspace at $12,000, 1 × onboarding package at $2,400, 8.875% tax."

> "Build a 5-slide quarterly review deck: Q1 revenue $5.1M (+34% YoY), top-3 wins, biggest risk, next-quarter goals."

> "Read `tasks.json` and produce an XLSX report grouped by assignee, with a pivot-style total row per person."

The agent picks the right tool, fills in the schema, and the server returns the file path.

## Artifact Management

Generated files are written to `PAPERJSX_OUTPUT_DIR` (defaults to OS temp). On every server start, artifacts older than `PAPERJSX_ARTIFACT_TTL_HOURS` are cleaned up. Each tool response includes the absolute path so clients can surface it or attach it as a resource.

## Security

- **SSRF protection** — remote URLs (image fetches, template references) pass through an allowlist that blocks RFC 1918 / 4193 ranges, cloud metadata endpoints (AWS / GCP / Azure), and localhost variants.
- **HTML escaping** — every tool that emits markup escapes user input (`& < > " '`) before templating.
- **Zod validation** — every tool input is validated before an engine is invoked. Schema errors are returned as structured JSON-RPC errors with field paths.

## Error Handling

Tool errors are returned as structured MCP responses. Input validation errors include the failing field path (e.g. `items[0].unitPrice: expected number, received string`).

## Running the Binary Directly

```bash
npx -y @paperjsx/mcp-server
```

The binary speaks MCP over stdio — use any MCP client to connect. No HTTP port is opened.

## Links

- SKILL.md — agent-skill definition distributed with this package
- JSON schema reference — `references/json-schema.md`
- Docs: https://paperjsx.com/docs

## License

MIT. See `LICENSE`.
