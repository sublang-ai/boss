<!-- SPDX-License-Identifier: Apache-2.0 -->
<!-- SPDX-FileCopyrightText: 2026 SubLang International <https://sublang.ai> -->

# Development

## Running from Source

```bash
git clone https://github.com/sublang-ai/boss.git
cd boss
npm install && npm run build
npm link
```

`npm link` points the global `boss` command to this checkout. After code changes, rebuild with `npm run build` (or use `npm run dev` for watch mode).

## Dev Sandbox Image

CI publishes a development image when `image/` files change on `main` or via manual workflow dispatch. To use it:

```bash
boss init --image ghcr.io/sublang-ai/boss-sandbox:dev-latest
```

If you have already initialized, edit `~/.boss/config.toml`:

```toml
[container]
image = "ghcr.io/sublang-ai/boss-sandbox:dev-latest"
```

Then pull and restart:

```bash
podman pull ghcr.io/sublang-ai/boss-sandbox:dev-latest
boss stop && boss start
```

### Image Tags

| Tag | Description |
| --- | --- |
| `latest` | Latest release (default) |
| `<version>` | Pinned release (e.g. `0.3.0`) |
| `dev-latest` | Latest dev build (from `image/` changes or manual dispatch) |
| `<short-sha>` | Commit build (push or manual dispatch) |
