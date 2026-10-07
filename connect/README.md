# Connect: who may call a server, and with whose credential

A Plugport server has two access settings (`PUT …/servers/{server}/access`):

- **inbound**, who may call it: `public`, `plugport_key` (a `pp_sk_` key in `Authorization`), or `oauth` (MCP clients
  connect through the server's own OAuth authorization server: Claude, ChatGPT, Cursor, VS Code, Claude Code…)
- **upstream**, whose credential reaches your API: `none`, `owner` (the environment's credential), `end_user_key` (each
  end user's own API key) or `end_user_oauth` (each end user's grant from your API's own OAuth)

With `oauth`, how end users connect is set with `PUT …/servers/{server}/connect`. The files here are its bodies.

| File | Access | For | Plan |
|---|---|---|---|
| [`team-email-sign-in.json`](team-email-sign-in.json) | `oauth` × `owner` | Your team uses an internal server from Claude.ai: they sign in with a code sent to an allowed address, and calls use your credential. | all |
| [`customer-api-keys.json`](customer-api-keys.json) | `oauth` × `end_user_key` | Your customers connect with their own API key, which Plugport checks with `GET /me` and keeps sealed. | Starter |
| [`api-oauth.json`](api-oauth.json) | `oauth` × `end_user_oauth` | Your customers sign in with your API's own OAuth; Plugport keeps and refreshes their tokens. Register the `upstream_callback_url` the endpoint returns with your OAuth app. | Pro |
| [`trusted-clients.json`](trusted-clients.json) | any `oauth` | Let one more app (your own agent platform) ask end users to connect. | all |

Notes:

- Letting anyone connect to a server that calls with your credential (`oauth` × `owner` without email sign-in) is public
  use of it: it needs `allow_public_owner`, as making the server public does.
- Only trusted apps may ask end users to connect: Claude, ChatGPT, VS Code and Cursor by default, apps returning to
  the same device (desktop and CLI clients), and clients you create. `trusted_hosts` replaces the defaults, so list
  them too.
- `toolsets` are what the consent page offers; end users can narrow them.
- Changing these settings ends the connections they no longer allow, at once.
- `identity` is a JSON pointer into your API's answer naming the end user (`/id`, `/data/user/email`): it's how
  connections are listed and analytics count end users.
