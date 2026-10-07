# Invoices: a spec that curates its own tools

[`openapi.yaml`](openapi.yaml) is a small invoicing API whose spec tells Plugport how its operations should look as MCP
tools. Importing it into a Plugport server gives these tools:

| Operation | Tool | Starts | Toolsets | Why |
|---|---|---|---|---|
| `GET /invoices` | `list_invoices` | on | full, read-only, billing | `x-mcp` rewrites the description for agents |
| `POST /invoices` | `create_invoice` | on | full, billing | `x-mcp` names it |
| `GET /invoices/{id}` | `get_invoice` | on | full, read-only, billing | |
| `POST /invoices/{id}/send` | `send_invoice` | on | full, billing | `x-mcp` gives it a title |
| `GET /customers/{id}` | `get_customer` | on | full, read-only | |
| `DELETE /invoices/{id}` | `delete_invoice` | off | full | deletes start off |
| `GET /reports/legacy` | `legacy_report` | off | full, read-only | deprecated operations start off |
| `POST /admin/customers/{id}/purge` | `purge_customer` | off | full | `x-mcp-exclude` on its path |
| `GET /internal/metrics` | `internal_metrics` | off | full, read-only | internal paths start off |

Every tool is in `full`; reads (GET) are in `read-only`; `x-mcp` adds the rest. A client asks for toolsets in its URL
(`/mcp?toolsets=billing`), or an end user picks them when connecting over OAuth.

## The vendor extensions

On an operation, or on a path for all of its operations (the operation wins):

```yaml
x-mcp:
  name: create_invoice        # the tool's name (a-z, 0-9, _; up to 64)
  title: Send invoice         # what clients show
  description: …              # what agents read to choose the tool
  toolsets: [billing]         # toolsets besides full (and read-only for GET)
  exclude: true               # leave the operation out
x-mcp-exclude: true           # the same, shorter
```

Specs written for Speakeasy work unchanged: `x-speakeasy-mcp` (`name`, `description`, `scopes` as toolsets,
`disabled`) is read too; `x-mcp` wins over it.

## Rules

[`rules.json`](rules.json) are a server's rules: they include or exclude operations on every import, by path glob
(`*` one segment, `**` any), tag, method or operation ID glob, the last match winning. With them, the table changes:

| Tool | Starts | Why |
|---|---|---|
| `internal_metrics` | off | rule 1 (`/internal/**`) |
| `legacy_report` | off | rule 2 (tag `reports`) |
| `delete_invoice` | **on** | rule 3 includes it |

What decides whether a tool is on, first to last: your own change in Plugport, then the spec's extensions, then the
server's rules, then the defaults (admin, internal and private paths and tags, deprecated operations and deletes start
off). Every tool says why it's off.

## A policy: which calls go through

[`policy.json`](policy.json) is the server's policy: rules checked in order on every tool call, the first whose
[CEL](https://cel.dev) condition holds allowing or denying it, else the default. Here:

| Rule | Does |
|---|---|
| `staff` | Acme staff who signed in with their email may call anything. |
| `no deletes` | Nobody else deletes (`tool.destructive`); agents read the message instead. |
| `big invoices` | Invoices over 10,000 (`args.amount` is in cents) need a person. |
| `evening sends` | Runs dry: sends after 18:00 Berlin time are recorded as "would deny", and go through. Turn it on once its numbers look right. |

Conditions read:

| | |
|---|---|
| `tool` | `name`, `title`, `toolsets`, `method`, `path`, `read_only`, `destructive`, `idempotent`, `open_world` |
| `args` | the call's arguments, as the tool's input schema has them |
| `caller` | `kind` (`public`, `key` or `connection`), `key` (`key_…`), `connection` (`con_…`), `via` (how the end user connected: `anyone`, `email`, `key` or `upstream_oauth`), `subject` (their email, or who they are at your API), `domain` (the email's domain), `client` (the MCP client's own name, not proof of who calls), `ip` |
| `server` | `slug`, `version`, `environment` |
| `now` | the time: `now.getHours("Europe/Berlin")`, `now.getDayOfWeek("UTC")` |

with CEL's string functions, `ip()` and `cidr()` (`cidr("10.0.0.0/8").containsIP(caller.ip)`), and optional values
(`args.?amount.orValue(0)`). A condition that can't be evaluated, say a missing argument, holds for a deny rule and not
for an allow rule: an error never lets a call through. Tools a caller can never call are left out of the tool list
they see.

The tests are calls and what the policy must decide for them; Plugport runs them on the server's tools whenever the
policy is saved, and refuses a policy whose tests fail. Try changes first with `POST …/servers/{server}/policy/test`,
which shows each rule checked.

## Re-imports keep your work

Import the spec again after it changes and new operations become tools, removed ones are flagged, and changed ones are
refreshed. A tool keeps its name (agents and configs depend on it), and what you changed in Plugport wins over the spec.
