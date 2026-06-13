# Bootstrap Workflow

Use this workflow when the user is in a new environment, asks to initialize Aclip, or `aclip` may not be installed.

First check whether the CLI exists:

```bash
command -v aclip
```

If `aclip` exists, verify it before continuing:

```bash
aclip --version
```

If `aclip` does not exist, check whether `pipx` exists:

```bash
command -v pipx
```

If `pipx` does not exist, tell the user to install it and reopen the terminal:

```bash
python3 -m pip install --user pipx
python3 -m pipx ensurepath
```

If the user is on macOS and Homebrew is available, `brew install pipx` is also acceptable, but do not require Homebrew.

After `pipx` is available, install the current release wheel:

```bash
pipx install https://github.com/likaihz/aclip/releases/download/v0.1.2/aclip-0.1.2-py3-none-any.whl
aclip --version
```

For local development from this repository, install from the Python package directory:

```bash
pipx install ./cli --force
aclip --version
```

Do not install from the repository root. The repository root is not the Python package root; the Python package lives under `cli/`.

After the CLI is installed and verified, initialize the user's vault and prove the first-resource loop:

```bash
aclip init-vault <path> --set-default
aclip doctor
aclip add http://example.com --later
aclip show <id>
```

Use the `id` returned by `aclip add` for `aclip show`. If any command returns JSON with `next_step` or `next_steps`, report those exact instructions before adding explanation. Do not move on to WeChat, Zhihu, or OpenCLI setup until generic first-resource capture works unless the user explicitly asks for protected-site setup first.
