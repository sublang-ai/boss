<!-- SPDX-License-Identifier: Apache-2.0 -->
<!-- SPDX-FileCopyrightText: 2026 SubLang International <https://sublang.ai> -->

# Environment: package installation

No `sudo` required. All package managers are pre-configured to install into
`~/.local/` via environment variables. Commands work directly:

```sh
pip install <pkg>
npm install -g <pkg>
cargo install <pkg>
go install <pkg>@latest
```

For standalone CLI tools (not language libraries), prefer mise:

```sh
mise use -g npm:<tool>
mise use -g github:<org>/<repo>
```

## Python virtualenvs

`PIP_USER=1` is set globally so that `pip install` works without `sudo`.
This conflicts with virtualenvs. After activating a venv, unset it:

```sh
source venv/bin/activate
unset PIP_USER
```

Interactive Bash shells handle this automatically via a `PROMPT_COMMAND`
hook. Non-interactive processes, non-Bash shells, and compound commands
(e.g., `source venv/bin/activate && pip install X`) must unset it explicitly.
