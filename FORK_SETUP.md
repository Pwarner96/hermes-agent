# Hermes Agent fork setup

This repository is a fork of [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent). The verified local runtime uses the upstream **v0.22.5** release (`bf0203a794a07aac3ff764972ab9087baa4e4f4e`). The fork's `main` branch retains the newer upstream snapshot plus this setup guide; the local runtime is checked out at the release tag.

## Start Hermes

```sh
hermes --cli
```

The default inference provider is `openai-codex` and the default model is `gpt-6.1-sol`, matching the existing Codex configuration. The existing ChatGPT/Codex login was imported through Hermes' model setup flow. No API key is needed for this connection.

Other entry points:

```sh
hermes --tui
hermes dashboard --no-open
hermes --version
hermes status
hermes skills list
```

The dashboard defaults to localhost on port 9119. Its launch command above does not open a browser automatically.

## Recreate the source installation

Use the official [installation guide](https://hermes-agent.nousresearch.com/docs/getting-started/installation). The tested source checkout and installer stages were:

```sh
git clone --depth 1 https://github.com/Pwarner96/hermes-agent.git
cd hermes-agent
git remote rename origin fork
git remote add origin https://github.com/NousResearch/hermes-agent.git
git config remote.pushDefault fork
git fetch --depth 1 origin tag v0.22.5
git switch --detach v0.22.5

bash scripts/install.sh --dir "$PWD" --stage venv --non-interactive
bash scripts/install.sh --dir "$PWD" --stage python-deps --non-interactive
bash scripts/install.sh --dir "$PWD" --stage config --non-interactive
bash scripts/install.sh --dir "$PWD" --stage products --non-interactive
```

The installer manages the pinned Python environment, Node.js, npm, ripgrep, FFmpeg, Chromium, agent-browser, computer-use driver, CLI launchers, terminal interface, web interface, and bundled skills. Use Hermes' package manager for later dependency changes; preserve its committed lockfiles.

To connect an existing Codex login on another machine:

```sh
codex login status
hermes model
```

Select OpenAI / ChatGPT or Codex Subscription, then accept the existing-credential import when offered. Use `hermes auth add openai-codex` when a fresh, separate login is desired. Set the model explicitly if needed:

```sh
hermes config set model.default gpt-6.1-sol
```

## Local state and updates

- User configuration: `~/.hermes/config.yaml`.
- Credentials: `~/.hermes/auth.json` and `~/.hermes/.env`, created with owner-only permissions.
- Sessions, memory, skills, logs, and managed dependencies: `~/.hermes/`.
- PATH launcher: `~/.local/bin/hermes`.
- `origin` points to the official repository so the runtime's default update channel follows stable releases.
- `fork` points to this repository; `remote.pushDefault=fork` directs ordinary pushes to the personal fork.

```sh
hermes update
hermes pm doctor
hermes doctor
```

Back up `~/.hermes/` before changing release channels or downgrading. The verified release can be selected again with `git switch --detach v0.22.5`; repeat the installer stages to rebuild the matching runtime. Older code may not understand state created by a newer release.

## Verification on 2026-10-31

- `hermes --version`: v0.22.5, stable, Python 3.14.9; reports up to date.
- Official dependency and product installation stages: exit 0.
- `hermes config check`: exit 0.
- `hermes pm doctor`: exit 0; default runtime packages installed. Optional packages are listed separately.
- `hermes computer-use status`: pinned driver 0.21.0 installed; runtime contract ready.
- Model smoke test through the existing Codex login returned exactly `HERMES_READY`.
- Agent tool smoke test used its terminal tool to query GitHub and returned `Pwarner96/hermes-agent`.
- Bundled skills are installed and list successfully.

## Known limits

The full `hermes doctor` exits 1. It reports unconfigured optional providers and upstream npm lockfile advisories: six findings for the web workspace and two for the TUI workspace when build dependencies are included. Production-only audits report **zero for web** and **one affected TUI package, `undici` 6.29.0**. npm reports a compatible fix at 6.29.1. See the [undici advisory](https://github.com/advisories/GHSA-2m5v-pch6-74hx). The classic CLI was used for the verified model and GitHub tests.

The installer timed out while registering the optional macOS computer-use app with Launch Services. A subsequent package status check passes, but screen control and macOS permission grants were not exercised.

Messaging services, scheduled jobs, external API integrations, and a desktop app build were not configured as part of this setup. No application source or dependency lockfiles were changed.
