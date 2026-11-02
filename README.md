# dsh-superagent

> **Research prototype for reference only, not an official DeepSeek product.** This code grew out of our team's day-to-day development work, with much of the engineering done by coding agents. Treat it as research code, not production-ready software. It may still contain bugs, and its coding style and engineering practices may be inconsistent. Review and adapt it before using it in your own environment.
>
> We have adapted it for publication, so it is not identical to our internal version or the implementation described in *Exploring Super-agent Design: Towards a More Autonomous Agent System*. Some designs discussed in the article are directions for future development and are not implemented in this prototype.

This repository is a fork of [deepseek-ai/dsh-superagent-prototype](https://github.com/deepseek-ai/dsh-superagent-prototype). It keeps the upstream design and adds the changes under [Fork notes](#fork-notes--本分支说明).

dsh-superagent is a proactive coding agent for one GitHub repository, packaged as a bundle for [DeepSeek Harness](https://www.deepseek.com/en/harness/). It listens to GitHub and, optionally, Feishu. When someone allowed asks the agent for work, it opens a work item, works on it in its own agent session and directory, and answers in the original conversation, for example with a pull request. A web panel shows what needs a person.

It is also a way of working with an agent: **take as little human attention as possible, and give as much value as possible.** An agent that can act for a whole team can also waste a whole team's attention, so it acts only when asked or when a rule the team wrote allows it, and speaks only when someone must act or can use a result. [docs/000-philosophy.md](docs/000-philosophy.md) explains the principle and what it asks of the operator.

## Install

dsh-superagent is a DSH bundle: `dsh plugin` installs it into one profile, next to the plugins that profile already loads.

### Requirements

- [DeepSeek Harness](https://www.deepseek.com/en/harness/) 0.2.1-alpha.2 or later. The desktop app 0.2.1-rc.5 is known to work.
- `gh` 2.48 or later and `git` 2.32 or later on `PATH`: the GitHub Source calls `gh`, and ticket checkouts use `git`.
- Optional, only for the Feishu Source: `lark-cli` and a Feishu or Lark app with a bot.

Node.js 22.19+ or 24+ is needed only when you build from source; the release tarball ships the built `lib/`.

### From the release (recommended)

```sh
dsh plugin --profile <profile> add \
  https://github.com/jwangkun/dsh-superagent/releases/latest/download/dsh-superagent.tgz
```

### From a local tarball

```sh
dsh plugin --profile <profile> add ./dsh-superagent-<version>.tgz
```

### Restart and open the panel

Restart that profile — quit and reopen the desktop app, or restart `dsh web` — then choose **Superagent** in the sidebar.

The bundle ships a default state directory, `<DSH_HOME>/state/dsh-superagent`, for its database, locks, worker shims and ticket directories. To keep it elsewhere, add this to the profile's `cordis.patch.yml` before the first start:

```yaml
- id: dsh-superagent
  config:
    stateDir: /absolute/path
```

`stateDir` cannot change after the first start.

### Configure it

Open **Superagent** and fill in:

1. **Sources → GitHub**: the repository as `owner/name`, and the `gh` CLI the agent acts as. The agent comments, pushes and opens pull requests as that account, so it needs write access; a dedicated account keeps its work distinguishable from yours.
2. **Policy**: the GitHub logins allowed to ask for work. Until the list has someone, no request can become a ticket.
3. **Execution**: the ticket directory root and a `prepare` command that gives each ticket its own checkout, for example a `git worktree` of a local clone.

Then ask for work from an allowed account: open an Issue or comment on one, mentioning the agent, for example `@<agent-login> fix the typo in the README`. With [shared-account mode](#shared-account-mode), the comment must *start* with the mention.

[docs/006-deployment.md](docs/006-deployment.md) covers the accounts, the `prepare` command template, upgrades and troubleshooting; [SETUP.md](SETUP.md) is the guided setup an agent can follow with you.

## Setup

Setup has many steps, so an agent does it with you. Ask your coding agent:

> Read SETUP.md in this repository and set up dsh-superagent with me.

It asks one short question at a time, shows the full plan once, and then installs and configures everything. [SETUP.md](SETUP.md) is what it follows.

## Documentation

Read the numbered documents in order; each builds only on the ones before it.

- [000 Philosophy](docs/000-philosophy.md)
- [001 Domain model](docs/001-domain.md)
- [002 Authorization](docs/002-authorization.md)
- [003 Execution](docs/003-execution.md)
- [004 Prompts](docs/004-prompts.md)
- [005 Panel](docs/005-panel.md)
- [006 Deployment](docs/006-deployment.md): manual setup, configuration, upgrades and troubleshooting
- [007 Architecture](docs/007-architecture.md)
- [008 Cookbook](docs/008-cookbook.md): adding a Source

## Fork notes / 本分支说明

This copy adds two changes on top of the upstream research prototype, for using it inside a shared DSH profile with one GitHub account:

- **One account can request and act.** A message the agent's own account writes counts as that account's explicit request when it *starts* with `@<login>`; its other messages (acknowledgements, progress, results) are still ignored, so the agent cannot trigger itself in a loop.
- **The worker environment stays out of the host process.** `PATH`, `GIT_CONFIG_GLOBAL` and `GIT_CONFIG_NOSYSTEM` are applied only for the setup check and restored at once. Installing the bundle into a profile you also use for other Sessions no longer changes their git or `PATH`; worker sessions use the operator's own CLI and git configuration, which one shared account makes equivalent.

### Shared-account mode

When the agent and the people asking share one GitHub account, mentioning yourself sends no notification, so the Source cannot rely on the account's notifications. Turn on **Receive new Issues** and **Receive comments in the repository** on the GitHub Source; it then reads the repository directly, and a comment that starts with the agent's mention becomes an explicit request. Messages of that account that do not start with the mention are treated as the agent's own output and ignored, which keeps acknowledgements and results from triggering new tickets.

## Development

`pnpm check` runs the typecheck, the tests and the build.

## License

[MIT](LICENSE)
