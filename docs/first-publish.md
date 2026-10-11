# Your First Publish

This walkthrough publishes a sample into the included mock backend’s in-memory inventory. It demonstrates the MCP workflow without changing a real website. The mock does not serve the preview or public pages named in its responses; restarting it discards your draft and publication.

For a real Laravel site, continue with the [Laravel first-publish guide](https://github.com/n2ns/laravel-post2site/blob/main/docs/first-publish.md) after this walkthrough. Your host adapter defines the real content fields and publication behavior.

## 1. Start the Local Backend

Use Node.js 24, npm, and Git. These shell commands use Bash syntax:

```bash
git clone https://github.com/n2ns/n2n-post2site.git
cd n2n-post2site
npm ci
npm run build
API_KEY=demo-key PORT=8787 node examples/mock-backend/server.mjs
```

Keep this terminal running. In a second terminal:

```bash
curl --fail-with-body -H 'X-API-KEY: demo-key' http://127.0.0.1:8787/capabilities
```

Expect HTTP 200 and JSON containing `contract: "post2site-publishing"`, `host_profile: "mock-backend"`, and `workflow.supports_drafts: true`. If the port is occupied, use a different `PORT` and change every URL below to match.

## 2. Connect Your MCP Client

For a client that accepts a JSON `mcpServers` configuration:

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

Replace the path with your built checkout. Other clients have different configuration formats: use their stdio MCP settings with the same command, arguments, and environment values. The path and `node` must exist in the environment where the client launches commands. A Windows client cannot use a Linux checkout path as a native Windows path; run the entry in the matching environment. Likewise, `127.0.0.1` must reach the backend from the MCP process.

Reconnect the MCP server. It exposes tools and resources, not an HTTP MCP URL. `CONTENT_API_BASE_URL` is the website API endpoint used by the bridge, not the URL to register as a remote MCP server.

The steps below name MCP operations and their JSON arguments. Ask your client to perform them, or use its tool/resource interface. Resource access varies by client; if yours cannot read resources, do not invent replacement tools. Use a client surface that supports resource reads. [Client Smoke Testing](./CLIENT_SMOKE.md) describes verification targets, not a claim that every client version has been tested.

## 3. Read the Website’s Rules

Read these three MCP resources before preparing content:

| Resource | Expected mock response |
|---|---|
| `post2site://capabilities` | `host_profile: "mock-backend"`; types `guide` and `note`; locales `en` and `zh` |
| `post2site://site-context` | `site_name: "Mock Site"`, default locale `en` |
| `post2site://editorial-policy` | `policy_version: "mock-1"`; English title and content required |

Resource responses contain JSON text. Tool responses expose the backend JSON in `structuredContent` and as text; some clients show only the text. The fields below refer to that backend JSON.

Call `n2n_list_inventory` with `{}` to inspect the seeded inventory. Before writing, call `n2n_check_duplicates` with:

```json
{
  "mode": "create",
  "target_identifier": "first-publish-demo",
  "content_payload": {
    "type": "guide",
    "topics": ["getting-started"],
    "locales": {
      "en": {
        "title": "My first Post2Site draft",
        "content": "This is a local demonstration of the draft and publish workflow."
      }
    }
  }
}
```

On a fresh mock, expect `duplicate: false` and an empty `issues` array. If you already published this identifier, use a new identifier or restart the mock to reset all demo data. These content fields belong to the mock host; another website may require a different payload.

## 4. Validate Before Saving

Call `n2n_validate_working_draft` with:

```json
{
  "mode": "publish",
  "article": {
    "mode": "create",
    "target_identifier": "first-publish-demo",
    "content_payload": {
      "type": "guide",
      "topics": ["getting-started"],
      "locales": {
        "en": {
          "title": "My first Post2Site draft",
          "content": "This is a local demonstration of the draft and publish workflow."
        }
      }
    }
  }
}
```

Expect `publishable: true`, `blockers: []`, and `warnings: []`. This validates the working draft without creating a saved draft. Resolve blockers before continuing; a successful draft save alone is not publication approval.

## 5. Save, Validate, and Review the Draft

After reviewing the text, tell the assistant: **“Save this draft to the local mock backend; do not publish it yet.”** Call `n2n_create_draft` with the same JSON used for `n2n_check_duplicates` in step 3 (the article object, without a surrounding `article` key).

Expect `status: "draft"`, the saved `content_payload`, and a generated `draft_id` such as `draft_<uuid>`. The bridge adds a `resource_uri` and resource link. Read `post2site://drafts/<draft_id>` to check the saved text. Replace `<draft_id>` in the following arguments with the returned value.

Call `n2n_validate_draft`:

```json
{"draft_id":"<draft_id>","mode":"publish"}
```

Expect `publishable: true` with empty `blockers` and `warnings`. Then call `n2n_preview_draft`:

```json
{"draft_id":"<draft_id>"}
```

Expect a `preview_url` ending in `/preview/<draft_id>` and an English entry in `preview_urls`. In this mock, those URLs are metadata only and do not render pages. Review the saved payload instead. A real backend must supply its preview implementation, and you should inspect that preview before approving publication.

## 6. Confirm and Publish

Pause for a separate, explicit instruction naming this draft and destination, for example: **“Publish draft `<draft_id>` to the local mock backend now.”** Do not treat permission to save or preview as permission to publish.

Only then call `n2n_publish_draft`:

```json
{"draft_id":"<draft_id>","publish_confirmed":true}
```

Expect `status: "published"`, a `resource_id`, and `canonical_url: "http://127.0.0.1:8787/blog/first-publish-demo"`. The MCP schema rejects missing or false `publish_confirmed`. This flag carries the decision; the client and operator must ensure an actual confirmation occurred.

Verify the result by reading `post2site://inventory/resources/first-publish-demo`. Expect `item.status: "published"` and `item.display_label: "My first Post2Site draft"`. You can also inspect the mock API directly:

```bash
curl --fail-with-body -H 'X-API-KEY: demo-key' \
  http://127.0.0.1:8787/inventory/resources/first-publish-demo
```

This inventory read is the completion check. Opening the returned canonical URL is not: the mock has no public page renderer. Nothing has been deployed or published to the internet.

Press Ctrl+C in the backend terminal when done and disconnect the demo MCP entry in your client. To publish to your website, follow the [Laravel guide](https://github.com/n2ns/laravel-post2site/blob/main/docs/first-publish.md) or implement the [HTTP contract](./BACKEND_API.md), reconnect with the real API URL/key, and repeat discovery using that host’s schema.

## Troubleshooting

| Symptom | Check |
|---|---|
| `401 Unauthorized` | The bridge’s `CONTENT_API_KEY` must match the mock’s `API_KEY`; the curl request uses `X-API-KEY`. `demo-key` is only for the demo. |
| Connection refused | Keep the backend terminal running and check its port. Test the URL from the environment running the MCP process. |
| `404` on discovery | Use `http://127.0.0.1:8787` for the mock. A default Laravel installation uses `https://your-site.example/api/v1/mcp`. Do not append `/capabilities` to the configured base URL. |
| MCP server cannot start | Run `npm run build`, check the absolute `dist/index.js` path, and ensure the client can run the configured Node executable. |
| Validation blockers / HTTP 422 | Use the field paths in the returned blockers; do not copy the mock’s schema into an unrelated host. Never put server-owned fields such as `status` or `published_at` inside `content_payload`. |
| Mock preview/public URL does not open | Expected: the mock returns URL metadata but does not implement those pages. Verify the saved draft and published inventory resource instead. |
| Draft disappears | Restarting the mock clears all drafts, assets, and demo publications. Use the draft ID from the current run. |
| Laravel adapter error | Package installation alone does not map your site’s data. Bind your `Post2SiteAdapter` and implement the host methods described in the [Laravel guide](https://github.com/n2ns/laravel-post2site/blob/main/docs/first-publish.md). |
