1: set OS defaults:
https://github.com/ricbra/dotfiles/blob/master/bin/setup_osx
https://git.herrbischoff.com/awesome-macos-command-line/about/#developer

roles: 
- dotfiles /
- homebrew /
- yabai    /
- font     /
- names    /
- vscode   /
- terminal |
- asdf  https://galaxy.ansible.com/osx_provisioner/asdf
- jetbrains https://github.com/reimarstier/ansible-role-jetbrains_installer
- defaults


requires login to continue: 

- vscode to sync
- firefox for extentions and config
- Jetbrain IDEs for sync

 https://github.com/ublue-os/fleek
 https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html

https://github.com/ansible-macos/macos-playbook/tree/master/roles

files to restore: 
~/.config/*
~/.oh-my-zsh/*

apps: 
telepresence

https://pijul.org/
