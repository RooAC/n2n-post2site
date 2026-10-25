<p align="center">
  <img src="./assets/n2n-post2site-logo.png" width="128" alt="n2n-post2site logo">
</p>

# n2n-post2site

Give your AI client a draft, review, and publish workflow for your website.

> **Draft with AI. Publish with intent.**

n2n-post2site connects an MCP client to your website’s publishing API. The assistant reads your content rules, prepares a draft, checks it with your backend, and publishes only after you explicitly confirm. Your website keeps ownership of its content model, storage, and publication rules.

Start with the included in-memory backend to try the complete workflow without connecting a real site. For a Laravel site, add the companion backend package and an adapter for your own content model. This project is a stdio MCP bridge, not a hosted CMS or a website generator.

## Architecture

```text
AI IDE client
  -> MCP stdio
n2n-post2site
  -> HTTPS + X-API-KEY
website content API
  -> host adapter handles storage, policy, preview, publish
```

For Laravel sites, [`n2ns/laravel-post2site`](https://github.com/n2ns/laravel-post2site) is the first-party backend package for this contract.

## Workflow

The intended authoring loop is:

1. Read `post2site://capabilities`, `post2site://site-context`, and `post2site://editorial-policy`.
2. Inspect existing content with `n2n_list_inventory`, `post2site://inventory/stats`, and `n2n_check_duplicates`.
3. Draft locally in the chat/IDE. Use `n2n_validate_working_draft` for non-persistent backend validation.
4. After the draft is confirmed for remote saving, call `n2n_create_draft`.
5. Upload only the selected image with `n2n_upload_asset`, then update the draft if needed.
6. Validate and preview with `n2n_validate_draft` and `n2n_preview_draft`.
7. Publish only after explicit publish confirmation with `n2n_publish_draft`.

The MCP server does not define blog fields such as `type`, `topics`, `geo_tags`, `seo_keywords`, or `locales`. Those live inside the host-declared `content_payload` object and are validated by the backend.

## Quick Start: Try a Local Draft

Use Node.js 24 for this walkthrough (the package runtime supports Node.js 22+), npm, and Git. In a terminal:

```bash
git clone https://github.com/n2ns/n2n-post2site.git
cd n2n-post2site
npm ci
npm run build
API_KEY=demo-key PORT=8787 node examples/mock-backend/server.mjs
```

Keep that terminal running. In another terminal, check the backend:

```bash
curl --fail-with-body -H 'X-API-KEY: demo-key' http://127.0.0.1:8787/capabilities
```

Expect JSON with `contract: "post2site-publishing"` and `host_profile: "mock-backend"`.

For clients that accept a JSON `mcpServers` configuration, add the following entry. Replace the absolute path with your checkout’s `dist/index.js`; the client must run `node` in the same environment as that file and reach the backend at the configured URL.

```json
{
  "mcpServers": {
    "n2n-post2site": {
      "command": "node",
      "args": ["/absolute/path/to/n2n-post2site/dist/index.js"],
      "env": {
        "CONTENT_API_BASE_URL": "http://127.0.0.1:8787",
        "CONTENT_API_KEY": "demo-key"
      }
    }
  }
}
```

Other clients use their own configuration syntax; transfer the command, arguments, and environment values rather than pasting this JSON unchanged. Restart or reconnect the configured MCP server in your client.

**Continue with [Your first publish](./docs/first-publish.md)** for exact resource reads, tool arguments, expected responses, and the confirmation step. The mock only changes in-memory data: it does not publish a website, and its preview/public URLs are examples without rendered pages. Stop it with Ctrl+C when finished; restarting resets the data.

## Connect Your Own Website

For Laravel, follow the [Laravel first-publish guide](https://github.com/n2ns/laravel-post2site/blob/main/docs/first-publish.md). It covers installing the backend and binding the adapter that maps the publishing contract to your content model. For another framework, implement the [Backend API Contract](./docs/BACKEND_API.md).

Once the backend is ready, set `CONTENT_API_BASE_URL` to its protected API base, such as `https://your-site.example/api/v1/mcp`, and set `CONTENT_API_KEY` to its content API key. The local mock uses the root URL; Laravel’s default includes `/api/v1/mcp`. Keep real keys out of prompts, screenshots, and article content.

You can keep the local checkout configuration above, or use the published npm package by replacing the command and arguments with:

```json
{
  "command": "npx",
  "args": ["-y", "n2n-post2site"]
}
```

Keep the two environment values in the client’s server entry. The first `npx` launch needs access to the npm registry. A real publication also needs your backend’s preview, storage, and public rendering implementation; the MCP bridge does not provide those.

## MCP Resources

Static resources:

- `post2site://capabilities`
- `post2site://site-context`
- `post2site://editorial-policy`
- `post2site://inventory/stats`

Resource templates:

- `post2site://inventory/resources/{target_identifier}`
- `post2site://drafts/{draft_id}`

List tools include canonical resource URIs and MCP `resource_link` blocks when returned rows can be opened as resources.

## MCP Tools

Inventory:

- `n2n_list_inventory`
- `n2n_check_duplicates`

Drafting:

- `n2n_validate_working_draft`
- `n2n_list_drafts`
- `n2n_create_draft`
- `n2n_update_draft`
- `n2n_validate_draft`
- `n2n_preview_draft`

Assets and publishing:

- `n2n_upload_asset`
- `n2n_publish_draft`

See [Resources Reference](./docs/RESOURCES_REFERENCE.md) and [Tools Reference](./docs/TOOLS_REFERENCE.md) for examples.

## Security Boundaries

n2n-post2site should not expose:

- Delete operations.
- Direct database queries.
- Filesystem writes.
- Shell commands.
- Deployment or server operations.
- Product, pricing, payment, subscription, or account administration.
- Server-managed lifecycle fields such as `published_at`, `author`, or host markers.

The backend is responsible for authentication, validation, preview rendering, publication, audit rules, and server-managed fields.

## Development

```bash
npm run build
npm test
npm run smoke:mcp
npm run check
```

`npm run check` runs build, unit tests, built-surface checks, and the MCP protocol smoke test.

## Related Docs

- [Your First Publish](./docs/first-publish.md)
- [Backend API Contract](./docs/BACKEND_API.md)
- [Architecture](./docs/ARCHITECTURE.md)
- [Resources Reference](./docs/RESOURCES_REFERENCE.md)
- [Tools Reference](./docs/TOOLS_REFERENCE.md)
- [Client Smoke Testing](./docs/CLIENT_SMOKE.md)
- [Mock Backend Example](./examples/mock-backend/README.md)
- [Roadmap](./ROADMAP.md)
- [Changelog](./CHANGELOG.md)
- [Security](./SECURITY.md)

## License

Apache-2.0
