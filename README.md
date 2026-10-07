# ManyScripts Video Transcripts

A Claude plugin for the [ManyScripts](https://www.manyscripts.com) public video
transcript service. This repository is a published mirror of the plugin
folder in ManyScripts' main (private) repository, kept in step on each
version bump; it exists so the plugin can be submitted to and installed from
a public source, per the [Claude plugin directory](https://claude.ai/directory)
requirements.

## What it provides

- Remote MCP server: `https://www.manyscripts.com/api/mcp`
- Tool: `extractTranscript`
- Sources: public YouTube, TikTok, Instagram Reel, X/Twitter, Facebook, and Xiaohongshu (beta) video URLs
- Results: complete spoken transcript, creator caption, and available metadata
- Authentication: ManyScripts OAuth (signed-in Free and Pro accounts)

## Install

```
/plugin marketplace add anhng211/manyscripts-claude-plugin
/plugin install manyscripts-video-transcripts@manyscripts
```

## Package layout

- `plugin.json`: portable Agent Plugins manifest. OpenAI-specific presentation lives under `extensions.com.openai.interface`, which is what the current OpenAI docs ask for in a portable package.
- `mcp.json`: portable MCP server configuration (`streamable-http`).
- `.codex-plugin/plugin.json` and `.mcp.json`: the Codex compatibility overlay. Because `plugin.json` carries an inline `extensions.com.openai`, OpenAI reads the portable manifest and ignores the overlay; it is here for legacy Codex readers and for the local validator, which reads only this file. Keep its fields identical to the portable ones. OpenAI's own reference plugins (`openai/plugins`, Notion and Figma) use this layout with `"type": "http"` and an `oauth_resource`.
- `.claude-plugin/plugin.json`: the Claude plugin manifest. Claude reads `.mcp.json` and discovers `skills/` by convention, so the manifest carries identity only; keep its name, version and description in step with the others.
- `.claude-plugin/marketplace.json`: lists this repository as a Claude plugin marketplace (one plugin, at the repository root).
- `skills/manyscripts-transcribe-video/`: invocation and result-handling guidance, plus `agents/openai.yaml`.
- `assets/`: square ManyScripts icons (192 px composer icon, 512 px logo).

When the OpenAI portal registers the MCP server it issues an app or connector id. At that point add `.app.json` (`{"apps": {"manyscripts": {"id": "<id from the portal>"}}}`) and an `"apps": "./.app.json"` field in `.codex-plugin/plugin.json` and under `extensions.com.openai`, the way the Notion and Figma reference plugins do.

## Transport

`POST /api/mcp` answers authenticated JSON-RPC directly, which is the stateless form of the MCP Streamable HTTP transport OpenAI requires. `GET /api/mcp` still opens the legacy SSE stream for older clients. Both share one OAuth flow.

## Validate locally

With the [Claude Code CLI](https://code.claude.com/docs/en/setup) installed:

```bash
claude plugin validate . --strict
```

## Smoke test the live service

Before every submission or release, run an authenticated end-to-end check against `https://www.manyscripts.com/api/mcp` from a signed-in Free account: OAuth discovery, dynamic client registration, the PKCE authorization flow in a browser, then `initialize`, `tools/list`, and one `tools/call`. The smoke-test script lives in the main ManyScripts repository (`scripts/`), since it needs the project's dependencies; this repository only ships the plugin itself.

## License

Proprietary. See `plugin.json`.
