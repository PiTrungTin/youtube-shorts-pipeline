# Changelog

## [3.0.0] — 2026-04-02

Generic niche support, NewsAPI integration, and multi-platform scaffold.

### Added
- **`--niche` flag** (`draft`, `run`, `topics`) — supported values: `gaming`, `finance`, `fitness`, `tech`, `beauty`, `food`, `travel`, `general`. Injects a one-line niche context into the Claude script prompt and routes topic discovery to niche-relevant subreddits + NewsAPI queries.
- **NewsAPI topic source** — `pipeline/topics/newsapi.py`. Fetches top headlines for the active niche. Requires `NEWSAPI_KEY` in env or `config.json`; silently skipped if absent. API key sent via `X-Api-Key` header (not URL param) to avoid log leakage. Rank-based trending score (1.0 → 0.3 decay).
- **`--platform` flag** (`draft`, `run`) — `shorts` (default), `reels`, `tiktok`, `all`. Sets `platform_label` + `max_script_words` hint in the Claude prompt. All platforms share 9:16 portrait for now; `PLATFORM_CONFIGS` in `config.py` is the single place to add per-platform rendering differences.
- **`PLATFORM_CONFIGS`** dict in `config.py` — width, height, `max_script_words`, display label per platform.
- **`NICHE_TO_SUBREDDITS`** dict in `config.py` — niche → default subreddit list applied when no explicit subreddits are set in `config.json`.
- **`get_newsapi_key()`** helper in `config.py`.

### Changed
- `generate_draft()` signature extended: `niche` (default `"general"`) and `platform` (default `"shorts"`). Both are stored in draft JSON.
- `TopicEngine.__init__()` accepts `niche` param; applies niche-based subreddit + NewsAPI query defaults at init time.
- NewsAPI enabled by default when `NEWSAPI_KEY` is present (no manual `enabled: true` needed in config.json).
- README bumped to v3.0.0 with niche examples, platform section, NewsAPI docs, and aarees.com CTA.
- Removed esports-specific language from default examples.

## [2.1.0] — 2026-02-27

Security audit fixes ported to v2 modular architecture.

### Security
- **Fix TOCTOU race in credential file writes.** `write_secret_file()` in `pipeline/config.py` now uses `os.open()` with `0o600` mode to atomically create files with correct permissions, eliminating the brief window where credentials were world-readable. Also applied to `scripts/setup_youtube_oauth.py`.
- **Escape ffmpeg concat file paths.** Single quotes in file paths are now properly escaped in `pipeline/assemble.py` for the ffmpeg concat demuxer.
- **Pin all dependency versions.** `pyproject.toml` and `requirements.txt` now use compatible-release bounds (e.g., `anthropic>=0.39.0,<1.0`) to reduce supply-chain risk.

### Fixed
- **Clear error on expired OAuth token without refresh token.** `pipeline/upload.py` now raises a descriptive `RuntimeError` instead of silently attempting to use expired credentials.

### Added
- Security section in `README.md` documenting all hardening measures.
- `CHANGELOG.md` (this file).

## [2.0.0] — 2026-02-27

Major restructure: modular `pipeline/` package with new features.

### Added
- **Burned-in captions** — word-by-word highlight via ASS subtitles (Whisper word timestamps).
- **Background music** — bundled royalty-free tracks with automatic voice-ducking.
- **Topic engine** — discover trending topics from Reddit, RSS, Google Trends, Twitter, TikTok.
- **Thumbnail generation** — Gemini Imagen + Pillow text overlay, auto-uploaded.
- **Resume capability** — pipeline state tracked per stage, re-runs skip completed work.
- **Retry logic** — `@with_retry` exponential backoff on all API calls.
- **Structured logging** — file + console logging, `--verbose` for debug output.
- **Claude Max support** — use Claude CLI as alternative to API key.
- **78 tests** — comprehensive test suite across all modules.
- `pyproject.toml` with proper packaging.

### Security (carried forward from audit)
- Gemini API key sent via `x-goog-api-key` header, not URL query parameter.
- Sanitized API error responses — no credential reflection.
- YouTube OAuth scope narrowed to `youtube.upload` + `youtube.force-ssl`.
- Default upload privacy set to `private`.
- Prompt injection mitigation — snippet truncation (300 chars) + boundary markers.
- LLM output validation — type-checking on all draft fields.
- `.gitignore` covers credential files, `.env`, output directories.

## [1.0.0] — 2026-02-27

Initial release. Single-file pipeline: draft → produce → upload.
