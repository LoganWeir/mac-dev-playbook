# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Ansible playbook for automating Mac development environment setup. It's a fork of Jeff Geerling's original mac-dev-playbook, customized with chezmoi for dotfiles management and specific language runtime configurations.

## Common Commands

### Running the Playbook

```bash
# Install Ansible roles/collections first
ansible-galaxy install -r requirements.yml

# Run the full playbook (will prompt for sudo password)
ansible-playbook main.yml --ask-become-pass

# Run specific tagged tasks only
ansible-playbook main.yml -K --tags "dotfiles,homebrew"

# Available tags: dotfiles, homebrew, mas, dock, sudoers, terminal, osx, screenshots, extra-packages, languages, python, node, chezmoi, sublime-text, post
```

### Testing

```bash
# Run tests (requires Ansible and configured test environment)
cd tests
ansible-playbook test.yml

# Lint the playbook
ansible-lint main.yml

# Check YAML syntax
yamllint .
```

### Managing Configuration

```bash
# Create custom config (copy from default and modify)
cp default.config.yml config.yml

# Test playbook changes in check mode (dry run)
ansible-playbook main.yml --check -K
```

## Architecture

### Playbook Structure

The playbook follows a modular architecture:

1. **main.yml** - Entry point that orchestrates the entire setup process
   - Loads configuration from `default.config.yml` and optionally `config.yml`
   - Executes external roles (command line tools, homebrew, dotfiles, mas, dock)
   - Imports task files for specific configurations

2. **Configuration Hierarchy**
   - `default.config.yml` - Contains all default settings and package lists
   - `config.yml` (optional) - User overrides for any default settings
   - Variables cascade: playbook defaults → default.config.yml → config.yml

3. **Task Organization** (`tasks/` directory)
   - Each task file handles a specific aspect of Mac configuration
   - Tasks are conditionally executed based on `configure_*` variables
   - Key task files:
     - `chezmoi.yml` - Manages dotfiles via chezmoi instead of traditional symlinks
     - `languages.yml` - Sets up Python (via pyenv) and Node.js (via fnm)
     - `osx.yml` - Executes macOS-specific configurations
     - `screenshots.yml` - Configures screenshot behavior and location
     - `extra-packages.yml` - Installs packages from composer, gem, npm, pip

4. **External Dependencies**
   - Uses Ansible Galaxy roles/collections defined in `requirements.yml`
   - Key roles:
     - `elliotweiser.osx-command-line-tools` - Ensures Xcode CLT is installed
     - `geerlingguy.mac.homebrew` - Manages Homebrew packages and casks
     - `geerlingguy.dotfiles` - Traditional dotfiles management (supplemented by chezmoi)
     - `geerlingguy.mac.mas` - Mac App Store installations
     - `geerlingguy.mac.dock` - Dock configuration via dockutil

### Key Customizations in This Fork

1. **Chezmoi Integration**: Uses chezmoi for dotfiles instead of simple symlinks, initialized with https://github.com/LoganWeir/dotfiles.git
2. **Language Runtimes**: Automatically installs Python 3.14 (pyenv) and Node 24 (fnm) as defaults
3. **Package Managers**: Includes uv for fast Python package management
4. **Screenshot Configuration**: Custom screenshot directory and behavior settings

## Development Workflow

### Adding New Software

1. **Homebrew packages/casks**: Add to `homebrew_installed_packages` or `homebrew_cask_apps` in config.yml
2. **Mac App Store apps**: Add to `mas_installed_apps` with app ID and name
3. **Language-specific packages**: Add to `composer_packages`, `gem_packages`, `npm_packages`, or `pip_packages`

### Adding New Tasks

1. Create a new task file in `tasks/` directory
2. Import it in `main.yml` with appropriate conditional and tags
3. Add any new configuration variables to `default.config.yml`

### Customizing macOS Settings

The `.osx` dotfile (managed via dotfiles) contains macOS-specific settings. Additional OS configurations can be added to `tasks/osx.yml`.

## Important Notes

- **GitHub Authentication**: Required before running if using private dotfiles repo with chezmoi. Use `gh auth login` first.
- **Git Configuration**: Must configure git user before chezmoi initialization will work
- **Manual Steps**: Some items still require manual setup (see full-mac-setup.md for complete list)
- **Idempotency**: Playbook is designed to be run multiple times safely
- **Remote Mac Support**: Can manage remote Macs via SSH by modifying the `inventory` file