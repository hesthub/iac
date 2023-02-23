1: set OS defaults:
https://github.com/ricbra/dotfiles/blob/master/bin/setup_osx
https://git.herrbischoff.com/awesome-macos-command-line/about/#developer

2: install packages via Brew

3: link dotfiles

roles: 
- dotfiles
- homebrew
- terminal
- yabai
- jetbrains https://github.com/reimarstier/ansible-role-jetbrains_installer
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