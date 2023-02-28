1: set OS defaults:
https://github.com/ricbra/dotfiles/blob/master/bin/setup_osx
https://git.herrbischoff.com/awesome-macos-command-line/about/#developer

roles: 
- dotfiles /
- homebrew /
- yabai    /

- terminal |
- font https://github.com/fubarhouse/ansible-role-macfonts

- jetbrains https://github.com/reimarstier/ansible-role-jetbrains_installer
            https://github.com/alanquillin/ansible_install_jetbrains_apps
- ssh https://github.com/sylvainmetayer/dotfiles/tree/main/roles
- docker
- macos (defaults)
- vim/neovim

more https://github.com/theNewFlesh/dev_roles
https://github.com/ansible-macos/macos-playbook/tree/master/roles

cant config? 
- firefox ( login and sync )
- vscode ( login and sync )

files to restore: 
~/.config/*
~/.skhdrc
~/.yabairc
~/.oh-my-zsh/*
~/.zprofile
~/.zshrc
~/.zshenv
~/Brewfile
~/.fzf.zsh


apps: 
telepresence


defaults

# Show filename extensions by default
defaults write NSGlobalDomain AppleShowAllExtensions -bool true

# Disable "natural" scroll
defaults write NSGlobalDomain com.apple.swipescrolldirection -bool false