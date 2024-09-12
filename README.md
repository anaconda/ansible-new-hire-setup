# Ansible-New-Hire-Setup

The ansible playbook in this repository installs and sets up a variety of packages useful for engineering new hires. For a basic setup you need only to do steps 1-2. Step 3 is for users who want further customization beyond just package management.

If you want this tool to be usable anywhere, we suggest you fork it and populate it with your personal variables.

# Terminology

**Ansible** - Python-based tool for predictable configuration management

**Homebrew** - MacOS package manager similar to **apt** or **yum**

**Package/Cask** - Different types of Homebrew installable applications, functionally the same thing for our purposes

# Base Packages

The lists of default casks and packages installed for new-hires are found in: /roles/new-hires/tasks/main.yml.

Currently these are:

```yaml
base_cask_list:
  - iterm2 # Terminal Emulator
  - lastpass
base_package_list:
  - ansible # To run this playbook
  - gh # Github CLI tool
  - awscli # AWS API CLI tool
  - diff-so-fancy # Additional diff options with Git
  - docker
  - docker-compose
  - ghq # Additional Git CLI tool for managing repos
  - htop # Performance and process monitor
  - jq # Json parser
  - pre-commit # Runs hooks against Git commits to enforce standards
  - tmux # Virtual terminal tool
  - vault # Hashicorp's secret manager
  - yq # Yaml parser
```

These are common tools that most engineers will find useful including git tools, iTerm2, Docker, and aws-cli. Feel free to remove ones you don't want.

# 1. Prerequisites

A few things need to be done to use this playbook:

1. Clone this repo to the local machine (requires Github access and valid SSH key [uploaded to GH](https://docs.github.com/en/enterprise-cloud@latest/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) and [authorized for SSO](https://docs.github.com/en/enterprise-cloud@latest/authentication/authenticating-with-saml-single-sign-on/authorizing-an-ssh-key-for-use-with-saml-single-sign-on))

```
git clone git@github.com:anaconda/ansible-new-hire-setup.git
```

2. [Install Homebrew for Mac](https://docs.brew.sh/Installation)

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

3. Install Miniconda: https://docs.anaconda.com/miniconda/miniconda-install/

4. Inside this repo, use miniconda to create an [environment](https://docs.conda.io/projects/conda/en/stable/user-guide/concepts/environments.html) and install Ansible

```
cd ansible-new-hire-setup
conda create -p ./env
conda activate ./env
conda install -c conda-forge ansible
```

5. Install the Ansible community upstream repository

```
ansible-galaxy install -r requirements.yaml
```

# 2. First Run - Installing Packages and Casks

The first run of this playbook will setup the base packages and generate a uservars file.

To run it, from the base directory run:

```
ansible-playbook setup-macbook.yml
```

**If you don't want any customization, this is the place to stop.** All of the base packages should have been installed and you're ready to go!

If there are packages, config files, or other customizations you want, please read on.

# Additional Useful Commands

If you'd like to see what Ansible is doing under the hood, use the `-v` or `--verbose` tag which will outputs git-style diffs of each config change.

To install and update packages only `ansible-playbook setup-macbook.yaml --tags packages`

To install everything including all customization `ansible-playbook setup-macbook.yaml --tags all` (Please make sure you aren't going to overwrite any config files!)

# 3. (Optional) Further Shell and Package Customization

The playbook can optionally set up:

- [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh) - enhances base MacOS shell and allows plugins/themes
- [powerlevel10k](https://github.com/romkatv/powerlevel10k) - ZSH theme, requires oh-my-zsh.

## Configuring PowerLevel10k

After you install P10k, it must go through a setup process.

Once installed, restart iterm and run `p10k configure`

## Font installation

1. Answer with "Yes“ once asked if you want to install the Meslo LGS Nerd Font and quit iterm once the installation is complete
2. If opened: Restart Visual Studio Code to fix the terminal font

---

For infrastructure-focused roles, the following are also available:

- Kubernetes tooling - including kubectl, helm, and related plugins and helpers
- Terraform tooling - including tflint and tfenv, for infrastructure provisioning

You can also specify additional packages/casks here. To install any or all of these items, set the booleans to true in `uservars/{{ ansible_user_id }}.yml`, where the user_id is the same as your MacOS username. This will will have been created already on the first run of the playbook, if it didn't exist already.

Add your custom packages and casks to `user_package_list` and `user_cask_list`. By default it looks like this:

```yaml
ohmyzsh: "false"
p10k: "false"
k8s: "false"
terraform: "false"

user_package_list:
  - vscode
user_cask_list:
  - somecask
```

To install those, run

```
ansible-playbook setup-macbook.yml --tags custom
```

# 4. (Very Optional) Advanced Customization - Directory, Config, and File Templating

You can also define specific paths, template files, and configurations to be created and populated. This is useful for quickly templating such files as vscode, git, and shell configurations.

This supports:

* plain files (`user_file_list`)
* templates (`user_template_list`)
* downloads from remote URLS (`user_url_list`). Defaults to overwriting files every time.

**Beware: Running the below will overwrite files in the destination directories [defined here](uservars/template.yml#L15), potentially causing data loss!**

To run these tasks, enter:

```
ansible-playbook setup-macbook.yml --tags adv-custom
```

This will install all custom paths and templates. All of this in contained in your personal uservar file.
