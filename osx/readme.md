1: set OS defaults:
https://github.com/ricbra/dotfiles/blob/master/bin/setup_osx
https://git.herrbischoff.com/awesome-macos-command-line/about/#developer

roles: 
- dotfiles /
- homebrew /
- yabai    /
- font     /
- terminal |
  
- jetbrains https://github.com/reimarstier/ansible-role-jetbrains_installer
            https://github.com/alanquillin/ansible_install_jetbrains_apps
- macos (defaults)
- vscode (install common extensions to save time)

https://github.com/ansible-macos/macos-playbook/tree/master/roles


nothing to config
- ssh
- docker
- 
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

https://pijul.org/

defaults

# Show filename extensions by default
defaults write NSGlobalDomain AppleShowAllExtensions -bool true

# Disable "natural" scroll
defaults write NSGlobalDomain com.apple.swipescrolldirection -bool false


Keyboard => Text => Correct spelling automatically => false
Keyboard => Text => Capitalize words automatically => false
Keyboard => Text => Add period with double space => false

Disable Siri

Disable Dock

Disable hot corners

finder - disable alot