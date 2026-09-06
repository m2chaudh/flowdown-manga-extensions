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
      "kind": "novel",                           // optional: "manga" (default) | "novel"
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

### `kind` field and the single-registry decision (added task 13, 2026-09-06)

Mobile's Novels feature and the pre-existing Manga feature share this one
`index.json` — mobile's `NovelsSourcesTab` filters to `kind: novel` entries
client-side (`novelEntries()` in `lib/features/novels/registry/novel_registry.dart`),
while the Manga Sources screen implicitly wants `kind != 'novel'` (any entry
omitting `kind`, i.e. every pre-existing manga extension, defaults to
`'manga'`).

This was a real decision point, not a default: desktop's Rust
`RegistryEntry` (`src-tauri/src/manga/registry.rs`) derives plain
`#[derive(Deserialize)]` with **no** `#[serde(deny_unknown_fields)]`
anywhere in that struct or its parent `RegistryIndex` — confirmed by
grepping the whole file. serde's default behavior for a struct without
that attribute is to silently ignore any JSON key it doesn't recognize,
so adding `"kind": "novel"` to entries in the shared `index.json` cannot
break desktop's parse: it simply never reads the field (desktop has no
novel-reading feature yet) and treats those entries exactly like it
treats every other entry it doesn't specifically use. Verified by reading
the struct definition directly, not by running desktop's test suite
(out of scope for the mobile-side task that added this).

Given that, a separate `index-novels.json` would have added a second
registry client/fetch path (and a second thing to keep in sync) to solve
a problem that doesn't exist. Novel entries live in this same
`index.json`, distinguished purely by the additive `kind` field.

`min_app_version` gating caveat: as of this writing, neither
`lib/features/manga/registry/registry_client.dart` /
`lib/features/manga/ui/screens/manga_sources_screen.dart` nor
`lib/features/novels/ui/screens/novels_sources_tab.dart` on the mobile
side actually **compares** `min_app_version` against the running app's
version anywhere — the field is parsed into `RegistryEntry.minAppVersion`
(`lib/features/manga/registry/models.dart`) and then never read again.
Setting `min_app_version` on an entry today does not hide it from an
older build; it's forward-looking metadata for whenever that comparison
gets implemented. Flagging here so nobody assumes the four novel entries
added in task 13 (`min_app_version: "0.2.9"`, one version past the
`0.2.8` this repo's mobile app was on when they were added) are actually
gated from older installs — they are not, yet.

## Cloudflare-gated sources

Extensions opting into `needs_browser_session: true` (i.e. sources that hide behind Cloudflare Turnstile, such as Toonily, Bato.to, MangaPark) require a headless-browser layer FlowDown doesn't currently ship. The YAML for these sources stays in `extensions/` (e.g. `extensions/toonily.yaml`) but is **not listed in `index.json`** — Browse registry won't surface them, and installing one directly via "Install from URL" will produce a clear "not supported" error at first fetch.

Reviving CF-gated sources would need either a system-Chrome integration that survives the macOS-bundle launch (see git history of `src-tauri/src/manga/cf_bypass.rs` for prior attempts) or a `flaresolverr`-style sidecar.

## Novel sources (`kind: novel`)

**Shipped (in `index.json`):** `wuxiabox`, `fanmtl`, `royalroad`.

**Committed but not indexed:**

- `libread.yaml` — uses `fetch_strategy: webview_proxied` (its chapter
  reader 302-redirects to freewebnovel.com, which runs a Cloudflare JS
  challenge a headless client can't solve). On mobile, the *foreground*
  manga-reader UI has a `CloudflareResolverMount` that can solve this
  kind of challenge, but the *background leech* `Runtime` bootstrapped in
  `lib/main.dart` — the thing that actually downloads novel chapters —
  deliberately has no `CloudflareResolver` wired into it in v1. That
  makes every real libread chapter download fail while browse/search
  keep working, i.e. shipping-broken for a novel source's core job.
  Excluded from `index.json` for this reason (see the exclusion comment
  at the top of `extensions/libread.yaml` for the full detail). Re-add
  once a `CloudflareResolver` is shared into the leech runtime's
  bootstrap (or an equivalent mount reachable from background
  downloads) and verified with an on-device leech-through-CF run.
- `ranobes.yaml` — search works, but every detail and chapter-list page
  on this network returns an anti-bot interstitial (a fake HTTP 404
  carrying `.antibot-btn-success` markup) to this template's requests,
  confirmed live across retries and cookie jars. Detail/chapter
  selectors are unverified best-effort. Excluded from `index.json`
  until that wall is worked around.

Both excluded YAMLs are still exercised by the mobile repo's
`test/features/novels/novel_templates_test.dart` parse-all-templates
coverage — only their `index.json` membership (installability) is
withheld, not their schema correctness.

## Contributing

1. Add `extensions/<id>.yaml` matching the FlowDown extension schema (see `src-tauri/src/manga/extension.rs` in the main repo).
2. Compute the YAML sha256: `shasum -a 256 extensions/<id>.yaml`.
3. Append a record to `index.json`.
4. Open a PR.
