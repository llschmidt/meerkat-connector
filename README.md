# Meerkat MCP server

A prompt engineer you talk to. It writes prompts, sharpens yours, and keeps them on your shelf.

This repository documents the [Meerkat](https://getmeerkat.dev) MCP server, the one behind Meerkat's Claude connector. The production server is hosted at `https://getmeerkat.dev/api/mcp`. This repo exists so anyone (users, directory reviewers, the curious) can see which tools the server exposes, what data each one receives, and how sign-in works, without access to Meerkat's closed-source application code.

## What Meerkat is

Meerkat is a prompt engineer you talk to. Describe what you want and it asks the one or two questions that matter, then writes the prompt. Paste a prompt you already have and it returns a sharper version with a short critique of what changed. Roast mode gives a severity score, a one-line verdict and a rebuilt version. Everything can be saved to your Meerkat shelf, searchable from your AI client or at [getmeerkat.dev](https://getmeerkat.dev).

## Connect

Remote server, Streamable HTTP, OAuth sign-in. Nothing to install, and a free Meerkat account works.

**Claude (web and desktop)**
1. Open Settings, then Connectors, then **Add custom connector**.
2. Server URL: `https://getmeerkat.dev/api/mcp`
3. Sign in with your Meerkat account when asked.

**Claude Code**
```
claude mcp add --transport http meerkat https://getmeerkat.dev/api/mcp
```
Then run `/mcp` to sign in.

**Other MCP clients with remote servers and OAuth** (Cursor, VS Code, Cline and others)
```json
{
  "mcpServers": {
    "meerkat": { "url": "https://getmeerkat.dev/api/mcp" }
  }
}
```
The client opens a browser sign-in on first use.

Setup guide: [getmeerkat.dev/docs/claude](https://getmeerkat.dev/docs/claude). Official MCP Registry name: `io.github.llschmidt/meerkat`.

## Tools

The server registers six tools.

| Tool | Title | Type | What it does |
| --- | --- | --- | --- |
| `chat_with_meerkat` | Chat with Meerkat | writes | Writes or revises a prompt from a short conversation about it, asking the questions that matter first. Saves the result to the user's shelf. |
| `refactor_prompt` | Sharpen a prompt | writes | Takes a prompt the user already has and returns a tighter version, with a short critique of what was strong and what changed. Saves the request and the revised prompt to the user's shelf. |
| `roast_prompt` | Roast a prompt | writes | Returns a severity from 0 to 5, a one-line verdict, the lines that hurt the prompt and why, and a rebuilt version, plus a link to a private page with the roast. Nothing is saved to the shelf. Limited to 5 roasts per hour per user. |
| `save_specimen` | Save to shelf | writes | Saves a prompt to the user's shelf, with an optional title, project and notes. |
| `search_shelf` | Search shelf | read-only | Searches the user's saved prompts by title or body text. |
| `list_projects` | List projects | read-only | Lists the user's project folders. |

No tool overwrites or deletes anything. Tools that write only create new rows in the signed-in user's own account. The full definition of each tool (description, input and output schemas, annotations), as the server returns it from `tools/list`, is in [`tools/`](./tools).

## What data leaves your client

When a tool is called, only that tool's arguments are sent to Meerkat. The server does not see the rest of your conversation.

| Tool | Data sent to Meerkat |
| --- | --- |
| `chat_with_meerkat` | The conversation turns about the prompt being worked on (your requests and Meerkat's earlier replies), and an optional working style. |
| `refactor_prompt` | The prompt text you want sharpened, optional notes on what to change, and an optional working style. |
| `roast_prompt` | The prompt text to roast. |
| `save_specimen` | The prompt you choose to save, and an optional title, project id and notes. |
| `search_shelf` | The search text, and an optional limit and project id. |
| `list_projects` | An optional limit. |

Meerkat does not train models on your prompts. See [`PRIVACY.md`](./PRIVACY.md) and [getmeerkat.dev/privacy](https://getmeerkat.dev/privacy) for storage, retention and deletion.

## How sign-in works

Meerkat uses OAuth with PKCE, with Supabase as the identity provider:

1. You add Meerkat to your MCP client.
2. The client sends you to `https://getmeerkat.dev/oauth/consent`.
3. You sign in with your Meerkat account, or create one.
4. The consent screen says in plain English what the client is asking to do.
5. On approval, the client receives a token scoped to your account.
6. Every tool call carries that token. The server resolves it to your account and limits all reads and writes to your own rows.

Discovery metadata is published at `https://getmeerkat.dev/.well-known/oauth-protected-resource` (RFC 9728).

## Source code

The MCP server, sign-in and web app live in Meerkat's private repository. This public repo contains only:

- setup and tool documentation (this README);
- tool definitions (`tools/`);
- the privacy notes (`PRIVACY.md`);
- the license (`LICENSE`).

The live server speaks the [Model Context Protocol](https://modelcontextprotocol.io) over Streamable HTTP at `https://getmeerkat.dev/api/mcp`.

## Support

- Setup help: [getmeerkat.dev/docs/claude](https://getmeerkat.dev/docs/claude)
- Support: [getmeerkat.dev/support](https://getmeerkat.dev/support), lateisha@schmade.com
- Built by [Schmade](https://schmade.com)

## License

[MIT](./LICENSE)
