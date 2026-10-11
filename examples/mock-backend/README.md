# Mock Post2Site Backend

Try the complete draft-to-publish API workflow without a website. This zero-dependency demo stores inventory, drafts, and asset records in memory; everything resets when it restarts. It does not render preview pages, public articles, or uploaded images. Returned URLs demonstrate the response shape only.

## Run from the Repository Root

Use Node.js 24 and run these commands in the `n2n-post2site` checkout:

```bash
npm ci
npm run build
API_KEY=demo-key PORT=8787 node examples/mock-backend/server.mjs
```

The server prints `Mock Post2Site backend listening on http://127.0.0.1:8787`. Leave the terminal open. The demonstration key is public; use this backend only for local experiments, not as a production service.

In another terminal:

```bash
curl --fail-with-body -H 'X-API-KEY: demo-key' http://127.0.0.1:8787/capabilities
```

Expect `contract: "post2site-publishing"` and `host_profile: "mock-backend"`. The API base is `http://127.0.0.1:8787`, without `/api/v1/mcp`. If port 8787 is occupied, choose another `PORT` and update the URL in both curl and the MCP client configuration.

## Connect and Publish a Demo Draft

Follow [Your first publish](../../docs/first-publish.md) for the local MCP configuration, all three discovery resources, exact draft payloads, validation, preview metadata, explicit publication confirmation, and inventory verification.

The mock demonstrates API-key authentication through `X-API-KEY`, inventory and duplicate checks, working-draft validation, saved drafts, asset records, and confirmed publication. Seeded inventory includes `welcome-guide` and `guides/example-post`; the latter can be read as `post2site://inventory/resources/guides%2Fexample-post`.

When finished, press Ctrl+C in the backend terminal and disconnect the demo MCP entry in your client. For a real Laravel site, use the [Laravel first-publish guide](https://github.com/n2ns/laravel-post2site/blob/main/docs/first-publish.md).

## Automated Protocol Check

From the repository root:

```bash
npm run smoke:mcp
```

This builds the bridge and starts its own temporary HTTP fixture. It checks the MCP surface and resource links, then closes its processes. It does not need the demo server above and does not establish that a particular IDE’s GUI has passed. See [Client Smoke Testing](../../docs/CLIENT_SMOKE.md).
