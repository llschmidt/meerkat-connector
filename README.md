# Meerkat — Claude Connector

The public surface of the Meerkat connector for Claude.

This repository documents the [Meerkat](https://getmeerkat.dev) MCP server that
powers Meerkat's Claude connector. The production server is hosted at
`https://getmeerkat.dev/api/mcp` and is the only endpoint Claude talks to —
this repo exists so anyone (Claude users, Anthropic reviewers, the curious)
can audit exactly what tools the connector exposes, what data it asks for,
and how the OAuth flow works, without needing access to Meerkat's
closed-source application code.

## What Meerkat is

Meerkat turns messy `"just do this for me"` requests into sharp, structured
prompts that work — without you learning prompt engineering. You write what
you mean; Meerkat does the engineering.

The Claude connector lets you use that same engine from inside Claude:

- Refine a messy prompt into something a model can actually run with
- Save anything from a Claude conversation to your Meerkat shelf at
  [getmeerkat.dev/workspace](https://getmeerkat.dev/workspace)
- Pull prompts you've already saved back into Claude when you need them again
- Keep your Meerkat projects in sync between the web app and Claude

## Tools the connector exposes

The MCP server registers five tools. JSON Schemas for each are in
[`tools/`](./tools).

| Tool                | Type      | What it does                                                                 |
| ------------------- | --------- | ---------------------------------------------------------------------------- |
| `chat_with_meerkat` | mutating  | Run a request through Meerkat's prompt-engineering pipeline. Auto-saves.     |
| `refactor_prompt`   | mutating  | Take an existing prompt and refine it. Auto-saves the refactored version.    |
| `list_projects`     | read-only | List the user's Meerkat project folders.                                     |
| `search_shelf`      | read-only | Search the user's saved prompts (title + body) on getmeerkat.dev.            |
| `save_specimen`     | mutating  | Explicitly save a prompt to the user's shelf. Optional project_id.           |

Every mutating tool is non-destructive: it can only create new rows, never
overwrite or delete existing ones. The user's existing shelf is never
mutated through this connector.

## How authentication works

Meerkat uses **OAuth 2.0 with PKCE**, backed by Supabase as the identity
provider:

1. The user adds Meerkat as a custom connector in Claude.
2. Claude redirects the user to `https://getmeerkat.dev/oauth/consent`.
3. The user signs in with their existing Meerkat / Supabase account
   (or creates one).
4. The consent screen shows exactly which scopes Claude is requesting,
   in plain English.
5. On approval, Claude receives a bearer token scoped to that user.
6. Every MCP call carries that token; the server resolves it to a
   Supabase user and scopes all reads and writes to that user's rows.

Discovery metadata is published at
`https://getmeerkat.dev/.well-known/oauth-protected-resource` per
RFC 9728.

## What data leaves Claude

When you call a tool, only the arguments you pass to that tool leave Claude.
The connector does not have access to the rest of your Claude conversation.

| Tool                | Data sent to Meerkat                                                     |
| ------------------- | ------------------------------------------------------------------------ |
| `chat_with_meerkat` | The `message` you send, optional `prompt_id`/`project_id`.               |
| `refactor_prompt`   | The `prompt` text you want refined, optional `notes`.                    |
| `list_projects`     | An optional name `query` and `limit`. Nothing about your conversation.   |
| `search_shelf`      | The `query` string and optional `project_id`.                            |
| `save_specimen`     | The `prompt` you choose to save, optional `title`, `project_id`, notes.  |

See [`PRIVACY.md`](./PRIVACY.md) for storage, retention, and deletion details.

## Setup (for end users)

1. Open Claude Desktop → Settings → Connectors → **Add custom connector**.
2. Server URL: `https://getmeerkat.dev/api/mcp`
3. Approve the OAuth flow.
4. In any Claude conversation, ask Claude to use Meerkat — for example,
   *"Use Meerkat to refine this prompt for me."*

Full setup walkthrough with screenshots:
[getmeerkat.dev/docs/claude](https://getmeerkat.dev/docs/claude).

## Source code

The MCP server, OAuth provider, web app, and Mission Control admin live in
Meerkat's private monorepo. This public repo intentionally contains only:

- Tool schemas (`tools/`)
- Privacy and terms (`PRIVACY.md`)
- Setup documentation (this README)
- License (`LICENSE`)

If you want to inspect the runtime behavior beyond what's documented here,
the live server speaks the [Model Context Protocol](https://modelcontextprotocol.io)
specification version `2025-06-18` over Streamable HTTP. Point any MCP
client at `https://getmeerkat.dev/api/mcp` after OAuth and call
`tools/list` for the canonical, machine-readable tool surface.

## Support

- Setup help: [getmeerkat.dev/docs/claude](https://getmeerkat.dev/docs/claude)
- Email: lateisha@schmade.com
- Built by [Schmade](https://schmade.com).

## License

[MIT](./LICENSE).
