# Sailwright Starter

A template for your team's machine setup. Fork it, pick your tools, and
preview the changes on macOS, Windows, or Linux before anything is installed.

Out of the box it installs Git, Python, Neovim, and jq and sets two shared Git
defaults. Change the list and you have your own team setup, versioned in Git
and testable in a throwaway VM.

## Before you start

- Install the [Sailwright CLI](https://github.com/csautter/sailwright#quick-start).
- On macOS, install [Homebrew](https://brew.sh).

## Quick start

### 1. Create your copy

Click **Use this template** on GitHub (or fork the repo), then clone your copy
and open a terminal in it.

### 2. Pick your tools

Edit the two lines in [`playbooks/team-setup.yml`](playbooks/team-setup.yml):

```yaml
team_defaults_tools: [git, python, neovim, jq]
team_defaults_git_config: { init.defaultBranch: main, pull.rebase: "true" }
```

The tools you can choose are listed in
[`roles/team_defaults/defaults/main.yml`](roles/team_defaults/defaults/main.yml):
`git`, `python`, `node`, `go`, `neovim`, `vscode`, `jq`, and `ripgrep`. See
[Add a tool](#add-a-tool) for anything else.

### 3. Preview the changes

Point Sailwright at this repo and run in check mode. Nothing is installed.

macOS and Linux:

```bash
export DEV_ALCHEMY_ROLE_SOURCES_CONFIG="$PWD/ansible-role-sources.yml"
sailwright provision local --check
```

Windows (PowerShell):

```powershell
$env:DEV_ALCHEMY_ROLE_SOURCES_CONFIG = "$PWD\ansible-role-sources.yml"
sailwright provision local --check
```

When the preview looks right, run the same command without `--check`.

Notes per OS:

- **Linux:** the preview on Debian and Ubuntu runs without `sudo`. The real
  run installs packages with `sudo`; if sudo asks for a password, add
  `--ask-become-pass` (short: `-K`).
- **Windows:** Sailwright sets up temporary local access for the run and asks
  for confirmation first. Start it from an elevated PowerShell to avoid the UAC
  prompt.

To keep the setting, add the `export` line to your shell profile (or set the
variable permanently on Windows).

## Test in a VM first

Try the setup on a clean machine before it touches your own. Keep the variable
from step 3 set; Sailwright uses the same playbook for VMs.

```bash
sailwright build list                      # what your host can run
sailwright build ubuntu --type server --arch amd64
sailwright create ubuntu --type server --arch amd64
sailwright provision ubuntu --type server --arch amd64 --check
sailwright provision ubuntu --type server --arch amd64
sailwright destroy ubuntu --type server --arch amd64
```

Replace `ubuntu --type server` with `windows11` or `macos` to test the other
platforms your host supports. The
[testing workflows guide](https://github.com/csautter/sailwright/blob/main/docs/testing-workflows.md)
lists every combination.

## What's inside

```text
.
├── ansible-role-sources.yml     # tells Sailwright where roles and playbooks live
├── playbooks/
│   └── team-setup.yml           # the team setup; your tools and Git defaults
├── roles/
│   └── team_defaults/           # example role, a pattern for your own roles
│       ├── defaults/main.yml    # tool catalog and default settings
│       ├── vars/main.yml        # package manager per OS
│       └── tasks/               # one file per OS family
└── .github/workflows/check.yml  # lints and applies the setup in an Ubuntu container
```

## Add a tool

Add one line to `team_defaults_catalog` in
[`roles/team_defaults/defaults/main.yml`](roles/team_defaults/defaults/main.yml)
with the package name for each package manager, then add the tool to
`team_defaults_tools`:

```yaml
kubectl: { brew: kubectl, choco: kubernetes-cli }
```

Leave out a package manager when the tool has no package there. The role then
skips the tool on that OS and says so in the output.

## Add your own role

Copy `roles/team_defaults/` to `roles/<your_role>/`, keep the files you need
(`tasks/main.yml` is the only required one), and add the role to the `roles:`
list in `playbooks/team-setup.yml`. Sailwright finds every role in `roles/`
through `ansible-role-sources.yml`, so no other configuration is needed. Start
`tasks/main.yml` with an include per OS family, as `team_defaults` does, when
the steps differ between macOS, Windows, and Linux, and run
`sailwright provision local --check` to try it.

## Share the setup with your team

Once your copy is on GitHub, teammates don't need to clone it. They put this in
Sailwright's `ansible-role-sources.yml` in their config folder (see
[Ansible role sources](https://github.com/csautter/sailwright/blob/main/docs/ansible-role-sources.md#config-location)):

```yaml
playbook: team-setup.yml
include_default_playbooks: false
sources:
  - name: team
    type: git
    url: https://github.com/<your-org>/<your-repo>.git
    ref: main
    roles_path: roles
    playbooks_path: playbooks
```

From then on, `sailwright provision local` pulls the latest version before each
run.

## Continuous integration

[`.github/workflows/check.yml`](.github/workflows/check.yml) runs on every push:
it lints the roles, previews and applies the setup in an Ubuntu container, and
runs it a second time to make sure nothing changes. You can also run the
playbook directly with Ansible:

```bash
ansible-playbook -i localhost, -c local playbooks/team-setup.yml --check
```

## License

[MIT](LICENSE). Use it as the base for your own setup without asking.
