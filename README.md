### Ansible User Customization Role
Install homebrew packages on OS-X and user-specific config files .vimrc, .screenrc, .bashrc, ssh configs and authorized keys on OS-X/Ubuntu


#### Variable Documentation
The following variables are used and should be set either in role defaults/main.yml or group_vars (described below)

```
# homebrew source code repository
homebrew_repo: https://github.com/Homebrew/brew

# path prefix for homebrew installation
homebrew_prefix: /usr/local

# homebreew install directory
homebrew_install_path: "{{ homebrew_prefix }}/Homebrew"

# path to homebrew executable
homebrew_brew_bin_path: /usr/local/bin

# local user for homebrew and config files (if running ansible as another user)
local_username: "{{ ansible_user_id }}"

# packages installed by default
homebrew_common_packages:
  - ssh-copy-id
  - tmux
  - screen
  - python
  - git
  - nmap
  - lftp
  - ipmitool
  - ansible
  - unrar
  - p7zip
  - minicom

# packages installed for specific user (defined in group_vars)
homebrew_user_packages: []

# blacklisted packages
homebrew_uninstalled_packages: []

# homebrew updates all existing packages on run
homebrew_upgrade_all_packages: no

# taps to be configured
homebrew_taps:
  - homebrew/core
  - homebrew/cask

# cask apps installed by default
homebrew_cask_common_packages:
  - 1password
  - bartender
  - synergy
  - osxfuse
  - google-chrome
  - slack
  - signal
  - spotify
  - sublime-text
  - xquartz
  - virtualbox
  - divvy

# cask apps installed for specific user (defined in group_vars)
homebrew_cask_user_packages: []

# blacklisted cask apps
homebrew_cask_uninstalled_packages: []

# directory to install cask apps
homebrew_cask_appdir: /Applications

# set up homebrew from brewfile
homebrew_use_brewfile: true

# location of brewfile
homebrew_brewfile_dir: '~'

```


#### group_vars/username/vars.yml
Uses group_vars to create user-specific .ssh/config, .bash_profile and ssh keys

```
---

homebrew_user_packages:
  - redis
  - telnet

homebrew_cask_user_packages:
  - steam


ssh_private_key: "{{ vault_ssh_private_key }}"
ssh_public_key: "{{ vault_ssh_public_key }}"

bash_profile: |

  # If you use macports
  if [ -d /opt/local/bin ]
  then
  export PATH=/opt/local/bin:/opt/local/sbin:/home/ldap/misc/openstack:/usr/local/Cellar:/usr/local/bin:$PATH
  fi

  # Shell color options
  export force_color_prompt=yes
  export CLICOLOR=1
  export LSCOLORS=ExFxcxdxcxegedbxgxEbEg

  if [ -x /usr/bin/dircolors ]; then
      test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
      alias ls='ls --color=auto'
      #alias dir='dir --color=auto'
      #alias vdir='vdir --color=auto'

      alias grep='grep --color=auto'
      alias fgrep='fgrep --color=auto'
      alias egrep='egrep --color=auto'
  fi

  # colored GCC warnings and errors
  export GCC_COLORS='error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01'

  # some more ls aliases
  alias ll='ls -l'
  alias la='ls -A'
  alias l='ls -CF'

  # Bash Prompt
  case $TERM in
  xterm*|xterm|linux)
  export PS1="\[\e]0;\h: \w\a\]\[\e[0;31m\]\h:\[\e[0;0m\]\w \[\e[0;37m\]\u\[\e[0;0m\]: "
  ;;
  *)
  export PS1="\h:\w \u\$ "
  ;;
  esac

ssh_config: |
  ForwardAgent yes
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  ServerAliveInterval 240
  TCPKeepAlive yes
  LogLevel QUIET
  User "{{ local_username }}"
  XAuthLocation /usr/X11/bin/xauth
```

#### group_vars/username/vault.yml
Encrypted entries for ssh keys, if desired.

```
ssh_private_key: |
  -----BEGIN RSA PRIVATE KEY-----
  ###############################
  -----END RSA PRIVATE KEY-----

ssh_public_key: >
  ssh-rsa #####################
```

#### inventory
create a new group in your inventory for your local machine that will inherit the group_vars created previously

```
[username]
localhost
```

#### playbook.yml
Sample playbook

```
---
- name: Set up user-specific packages/configs etc
  hosts: localhost
  connection: local
  become: False
  tasks:
    - import_role:
        name: user
```
