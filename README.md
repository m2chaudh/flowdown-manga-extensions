# FlowDown Manga Extensions

Curated registry of YAML manga source extensions for [FlowDown](https://github.com/m2chaudh/flowdown).

The FlowDown desktop app reads `index.json` from this repo on launch (and hourly thereafter) to surface installable / updatable extensions in Settings → Manga.

## Layout

```
index.json                      registry index
extensions/<id>.yaml            extension source files
```

## index.json schema

```jsonc
{
  "registry_version": "1.0",
  "extensions": [
    {
      "id": "mangadex",                          // unique, lowercase
      "version": "1.0.0",                        // semver; bump to trigger auto-update
      "api_version": 1,                          // FlowDown extension-runtime API version
      "min_app_version": "0.1.0",                // optional: minimum FlowDown version
      "name": "MangaDex",
      "language": "en",                          // ISO 639-1 or "multi"
      "nsfw": "optional",                        // one of: no | optional | yes
      "description": "Open scanlation catalog, ~80k titles.",
      "yaml_url": "https://raw.githubusercontent.com/m2chaudh/flowdown-manga-extensions/main/extensions/mangadex.yaml",
      "icon_url": "",                            // optional
      "sha256": "..."                            // optional integrity check on yaml_url body
    }
  ]
}
```

`sha256` is optional. When present, FlowDown verifies the fetched YAML matches before installing or updating. Empty = skip.

## Cloudflare-gated sources

Extensions opting into `needs_browser_session: true` (i.e. sources that hide behind Cloudflare Turnstile, such as Toonily, Bato.to, MangaPark) require a headless-browser layer FlowDown doesn't currently ship. The YAML for these sources stays in `extensions/` (e.g. `extensions/toonily.yaml`) but is **not listed in `index.json`** — Browse registry won't surface them, and installing one directly via "Install from URL" will produce a clear "not supported" error at first fetch.

Reviving CF-gated sources would need either a system-Chrome integration that survives the macOS-bundle launch (see git history of `src-tauri/src/manga/cf_bypass.rs` for prior attempts) or a `flaresolverr`-style sidecar.

## Contributing

1. Add `extensions/<id>.yaml` matching the FlowDown extension schema (see `src-tauri/src/manga/extension.rs` in the main repo).
2. Compute the YAML sha256: `shasum -a 256 extensions/<id>.yaml`.
3. Append a record to `index.json`.
4. Open a PR.
