# Plugport examples

[Plugport](https://plugport.ai) turns your OpenAPI spec into a hosted, OAuth-secured MCP server that agents actually use
well. These examples show the parts you write yourself.

| | |
|---|---|
| [`invoices/`](invoices) | A spec that curates its own tools with `x-mcp`, and server rules that include or exclude operations on every import. |
| [`connect/`](connect) | How end users connect over OAuth: email sign-in for your team, your customers' own API keys, or your API's own OAuth. |
| [`github-actions/`](github-actions) | Publish a new version on every release with [`plugport-ai/publish`](https://github.com/plugport-ai/publish), and check pull requests. |

## Connecting clients

Every server lives at `https://{slug}.plugport.run/mcp`. Opening its address in a browser shows how to add it to
Claude, ChatGPT, Cursor, VS Code, Claude Code, Codex, Gemini CLI, Cline, Zed, Devin Desktop and Goose, with buttons
and commands filled in. Your docs can link there, or show the badge:

```markdown
[![Connect Acme](https://acme.plugport.run/install/badge.svg)](https://acme.plugport.run/)
```

For example, with Claude Code:

```sh
claude mcp add --transport http acme https://acme.plugport.run/mcp
```

Plugport is in beta: [join it](https://plugport.ai).

## License

[MIT](LICENSE)
