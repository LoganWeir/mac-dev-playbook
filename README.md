<img src="https://raw.githubusercontent.com/geerlingguy/mac-dev-playbook/master/files/Mac-Dev-Playbook-Logo.png" width="250" height="156" alt="Mac Dev Playbook Logo" />

# Mac Development Ansible Playbook

[![CI][badge-gh-actions]][link-gh-actions]

This playbook installs and configures most of the software I use on my Mac for web and software development. Some things in macOS are slightly difficult to automate, so I still have a few manual installation steps, but at least it's all documented here.

## Installation

### Quick Start (Recommended Order)

  1. **Install Xcode Command Line Tools:**
     ```bash
     xcode-select --install
     ```

  2. **Sign in to the Mac App Store** (required for any App Store apps)

  3. **Install Homebrew:**
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

     # For Apple Silicon Macs (M1/M2/M3/M4):
     echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
     eval "$(/opt/homebrew/bin/brew shellenv)"

     # For Intel Macs:
     # Homebrew is usually already in PATH at /usr/local/bin
     ```

  4. **Install Ansible:**
     ```bash
     # Using pip (recommended)
     pip3 install --user ansible

     # Add to PATH if needed:
     export PATH="$HOME/Library/Python/3.9/bin:$PATH"
     ```

  5. **Clone this repository:**
     ```bash
     git clone https://github.com/LoganWeir/mac-dev-playbook.git
     cd mac-dev-playbook
     ```

  6. **Install Ansible requirements:**
     ```bash
     ansible-galaxy install -r requirements.yml
     ```

  7. **Create your config override file (optional):**
     ```bash
     cp default.config.yml config.yml
     # Edit config.yml to customize your setup
     ```

  8. **Run the playbook:**
     ```bash
     ansible-playbook main.yml --ask-become-pass
     ```

### GitHub Authentication (Only if using private dotfiles repo)

If your chezmoi dotfiles repository is private, authenticate with GitHub first:

```bash
# Install GitHub CLI (if not already installed via playbook)
brew install gh

# Authenticate with GitHub
gh auth login
# Choose: GitHub.com -> HTTPS -> Login with web browser

# Setup git to use GitHub CLI for authentication
gh auth setup-git

# Configure git identity
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

> Note: If some Homebrew commands fail, you might need to agree to Xcode's license or fix some other Brew issue. Run `brew doctor` to see if this is the case.

### Use with a remote Mac

You can use this playbook to manage other Macs as well; the playbook doesn't even need to be run from a Mac at all! If you want to manage a remote Mac, either another Mac on your network, or a hosted Mac like the ones from [MacStadium](https://www.macstadium.com), you just need to make sure you can connect to it with SSH:

  1. (On the Mac you want to connect to:) Go to System Settings > Sharing.
  2. Enable 'Remote Login'.

> You can also enable remote login on the command line:
>
>     sudo systemsetup -setremotelogin on

Then edit the `inventory` file in this repository and change the line that starts with `127.0.0.1` to:

```
[ip address or hostname of mac]  ansible_user=[mac ssh username]
```

If you need to supply an SSH password (if you don't use SSH keys), make sure to pass the `--ask-pass` parameter to the `ansible-playbook` command.

### Running a specific set of tagged tasks

You can filter which part of the provisioning process to run by specifying a set of tags using `ansible-playbook`'s `--tags` flag. The tags available are:

- `homebrew` - Install Homebrew packages and cask applications
- `mas` - Install Mac App Store applications
- `dock` - Configure the Dock
- `osx` - Configure macOS system settings
- `package-managers` / `packages` - Install packages via pip, npm, gem, etc.
- `languages` / `python` / `node` - Configure Python and Node.js environments
- `chezmoi` / `dotfiles` - Set up chezmoi for dotfiles management
- `zed` / `editors` - Configure Zed editor
- `post` - Run post-provisioning tasks

Example:

    ansible-playbook main.yml -K --tags "homebrew,osx"

## Overriding Defaults

Not everyone's development environment and preferred software configuration is the same.

You can override any of the defaults configured in `default.config.yml` by creating a `config.yml` file and setting the overrides in that file. For example, you can customize the installed packages and apps with something like:

```yaml
homebrew_installed_packages:
  - git
  - go

mas_installed_apps:
  - { id: 443987910, name: "1Password" }
  - { id: 498486288, name: "Quick Resizer" }
  - { id: 557168941, name: "Tweetbot" }
  - { id: 497799835, name: "Xcode" }

composer_packages:
  - name: hirak/prestissimo
  - name: drush/drush
    version: '^8.1'

gem_packages:
  - name: bundler
    state: latest

npm_packages:
  - name: webpack

pip_packages:
  - name: mkdocs

configure_dock: true
dockitems_remove:
  - Launchpad
  - TV
dockitems_persist:
  - name: "Sublime Text"
    path: "/Applications/Sublime Text.app/"
    pos: 5
```

Any variable can be overridden in `config.yml`; see the supporting roles' documentation for a complete list of available variables.

## What This Playbook Does

### System Configuration
- **macOS Settings**: Configures trackpad, keyboard, Finder, Dock, and system preferences
- **Power Management**: Sets sleep, standby, and display settings
- **Security**: Password requirements, Gatekeeper settings

### Development Environment
- **Package Managers**: Homebrew, pip, npm, gem
- **Languages**: Python 3.14 (via pyenv), Node.js 24 (via fnm)
- **Dotfiles**: Managed via [chezmoi](https://www.chezmoi.io/) with repository https://github.com/LoganWeir/dotfiles.git

### Default Applications (via Homebrew Cask)
- Development: Docker, ChromeDriver
- Browsers: Firefox, Google Chrome
- Communication: Slack
- Database: Sequel Ace
- Utilities: Handbrake, LICEcap, Transmit

### Default Packages (via Homebrew)
- Core tools: git, wget, openssl, ssh-copy-id
- Development: go, autoconf, doxygen
- Python: pyenv, uv (fast package manager)
- Node.js: fnm (Fast Node Manager)
- Database: sqlite, postgresql, postgis
- Network: nmap, httpie, iperf
- Shell: bash-completion, zsh-history-substring-search


## Post-Installation

After running the playbook:

- Import Rectangle configuration from `files/app_config/logan_rectangle_config.json`
- Import VSCode profile from `files/app_config/Logan.code-profile`
- Install Zed extensions (settings will be copied to `~/.config/zed/settings.json`)
- Install Chrome extensions: Vimium, Bitwarden, 1Password, Earth View from Google
- Install MCP servers in Claude Code: context7, figma, jira, github

## Full / From-scratch setup guide

Since I've used this playbook to set up something like 20 different Macs, I decided to write up a full 100% from-scratch install for my own reference (everyone's particular install will be slightly different).

You can see my full from-scratch setup document here: [full-mac-setup.md](full-mac-setup.md).

## Troubleshooting

### Common Issues

**Homebrew installation fails:**
```bash
brew doctor  # Check for issues
brew update  # Update Homebrew
```

**Ansible can't find Python:**
```bash
# Use explicit Python path
ansible-playbook main.yml --ask-become-pass -e ansible_python_interpreter=/usr/bin/python3
```

**Permission denied errors:**
- Make sure to use `--ask-become-pass` (or `-K`) when running the playbook
- Some macOS settings require Full Disk Access for Terminal

**Mac App Store apps fail to install:**
- Ensure you're signed in to the Mac App Store before running the playbook
- Some apps may require purchase or previous download

**Chezmoi fails to initialize:**
- If using a private repository, ensure GitHub authentication is set up first
- Check that git is configured with your user.name and user.email

## Testing the Playbook

### Local Testing
```bash
# Syntax check only
ansible-playbook main.yml --syntax-check

# Dry run (check mode)
ansible-playbook main.yml --check -K

# Run specific tags only
ansible-playbook main.yml -K --tags "osx"
```

### CI Testing
This project is [continuously tested on GitHub Actions' macOS infrastructure](https://github.com/geerlingguy/mac-dev-playbook/actions?query=workflow%3ACI).

### VM Testing
For testing in a virtual machine:
  - [UTM](https://mac.getutm.app) - Free, open-source
  - [Tart](https://github.com/cirruslabs/tart) - CLI-based

## Ansible for DevOps

Check out [Ansible for DevOps](https://www.ansiblefordevops.com/), which teaches you how to automate almost anything with Ansible.

## Author

This project was created by [Jeff Geerling](https://www.jeffgeerling.com/) (originally inspired by [MWGriffin/ansible-playbooks](https://github.com/MWGriffin/ansible-playbooks)).

[badge-gh-actions]: https://github.com/geerlingguy/mac-dev-playbook/actions/workflows/ci.yml/badge.svg
[link-gh-actions]: https://github.com/geerlingguy/mac-dev-playbook/actions/workflows/ci.yml
