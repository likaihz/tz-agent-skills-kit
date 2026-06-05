# CLI Contract

Use the `aclip` CLI for deterministic operations whenever a command exists. Do not edit Aclip notes by hand unless the CLI cannot perform the requested operation.

## Vault Setup

Before capture, ensure Aclip has a vault. If `aclip add`, `aclip list`, or `aclip show` reports that no vault is configured, ask for the intended vault path and run:

```bash
aclip init-vault <path> --set-default
```

After initializing the vault, run:

```bash
aclip doctor
```

If `aclip doctor` reports vault/config errors, stop and report `next_steps`; when doctor output has `next_steps`, report those exact steps. If `aclip doctor` only reports OpenCLI missing/disconnected, generic or direct captures may proceed. Report those `next_steps` only when the user is trying WeChat/Zhihu/protected-site capture.

## Dependency Commands

For WeChat Official Account and Zhihu capture, ensure OpenCLI is installed and connected:

```bash
aclip deps opencli install
aclip deps opencli check
```

If install returns `connected: false`, tell the user to confirm the OpenCLI Chrome extension install in the opened Chrome page, then rerun `aclip deps opencli check`.

After installing OpenCLI or resolving a connection issue, run `aclip doctor`. If it reports vault/config errors, stop and report `next_steps`. If it only reports OpenCLI missing/disconnected, generic or direct captures may proceed; report those `next_steps` only for WeChat/Zhihu/protected-site capture.

## Common Commands

```bash
aclip show <id>
aclip update <id> --priority high
aclip update <id> --class tool
aclip update <id> --usage reusable --topic "AI Agent" --tag important
aclip update <id> --state to_read
aclip update <id> --read-status reading
aclip mark-read <id>
aclip archive <id>
aclip retry <id> --capture-provider opencli
aclip delete <id>
```

## Error Handling

When `aclip add` or `aclip retry` returns an error, do not guess. If the error includes `next_step`, report that exact `next_step` field before adding your own explanation. If the error is environment-related or protected-site capture failed, run `aclip doctor`; when doctor output has `next_steps`, report those exact steps.

If doctor only reports OpenCLI missing/disconnected, generic or direct captures may proceed, but WeChat/Zhihu/protected-site capture needs those OpenCLI `next_steps` first.
