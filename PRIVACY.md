# Privacy — Meerkat Claude Connector

This document explains exactly what data the Meerkat Claude connector
receives, where it is stored, how long it is kept, and how to delete it.

For the broader Meerkat product privacy policy, see
[getmeerkat.dev/privacy](https://getmeerkat.dev/privacy).

## What we receive

The connector only receives the arguments of tool calls you (or Claude on
your behalf) make. It does **not** have access to the rest of your Claude
conversation. Claude only forwards the specific arguments declared in each
tool's schema.

| Tool                | Arguments forwarded to Meerkat                                                |
| ------------------- | ----------------------------------------------------------------------------- |
| `chat_with_meerkat` | `message`, optional `prompt_id`, optional `project_id`                        |
| `refactor_prompt`   | `prompt`, optional `notes`                                                    |
| `list_projects`     | optional `query`, optional `limit`                                            |
| `search_shelf`      | `query`, optional `limit`, optional `project_id`                              |
| `save_specimen`     | `prompt`, optional `title`, optional `project_id`, optional `notes`           |

We also log standard request metadata: the authenticated user id, the tool
name, a timestamp, and the size of the call. We do not log argument bodies
beyond what is persisted to your shelf as described below.

## What we store

When you call `chat_with_meerkat`, `refactor_prompt`, or `save_specimen`,
Meerkat creates a new row on your **shelf** at
[getmeerkat.dev/workspace](https://getmeerkat.dev/workspace). The row is
identical to one you would have created in the web app and is owned by your
Meerkat account. It contains:

- The prompt text
- A short title
- The tool that produced it (tagged `claude-chat`, `claude-refactor`, or
  `claude-save`, so you can tell at a glance where it came from)
- The optional project id you specified
- The timestamp

`list_projects` and `search_shelf` are **read-only**. They never create or
modify rows.

## Where data is stored

Meerkat stores all user data in [Supabase](https://supabase.com), a managed
Postgres provider. Data is encrypted at rest and in transit. Database access
is gated by row-level security: every connector call is scoped to the
authenticated user's `user_id`, and the server cannot read or write rows
belonging to anyone else.

## Retention

- Shelf rows: kept until you delete them, or until you delete your Meerkat
  account, whichever comes first.
- Authentication tokens issued to Claude: revocable at any time from
  [getmeerkat.dev/account](https://getmeerkat.dev/account) or from inside
  Claude (Settings → Connectors → Disconnect).
- Request metadata logs: kept up to 30 days for operational debugging,
  then deleted.

## Deletion

You have three independent ways to remove data:

1. **Delete a single shelf row.** Go to your shelf at
   `getmeerkat.dev/workspace`, open the row, click delete. This removes
   everything saved through the connector for that specimen.
2. **Disconnect the connector.** From inside Claude, disconnect Meerkat.
   This revokes Claude's access token and prevents future calls. Data
   already on your shelf is preserved (you can still see it on the web).
3. **Delete your Meerkat account.** Email lateisha@schmade.com from the
   email associated with the account. We delete all rows owned by that
   account, including everything created via the connector, within
   30 days.

## Third parties

The connector calls one external service when running
`chat_with_meerkat` or `refactor_prompt`: an LLM provider (currently
the [Vercel AI Gateway](https://vercel.com/ai-gateway)) used to run
Meerkat's prompt-engineering pipeline. The LLM provider receives the
prompt content for the duration of the request and does not retain it.

No data is sold, shared with advertisers, or used to train external
models.

## Contact

Questions, deletion requests, security disclosures:
**lateisha@schmade.com**
