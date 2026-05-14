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

## Contributing

1. Add `extensions/<id>.yaml` matching the FlowDown extension schema (see `src-tauri/src/manga/extension.rs` in the main repo).
2. Compute the YAML sha256: `shasum -a 256 extensions/<id>.yaml`.
3. Append a record to `index.json`.
4. Open a PR.
