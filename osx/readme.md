1: set OS defaults:
https://github.com/ricbra/dotfiles/blob/master/bin/setup_osx
https://git.herrbischoff.com/awesome-macos-command-line/about/#developer

roles: 
- dotfiles /
- homebrew /
- yabai    /
- font     /
- terminal |


 https://github.com/ublue-os/fleek
 https://www.redhat.com/sysadmin/developing-ansible-role
 https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#role-directory-structure
 https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html

 https://github.com/markosamuli/ansible-asdf/tree/master/tasks

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



# Firefox

validates that firefox is install
generate profile and install extensions

https://github.com/unrblt/ansible-role-firefox

https://support.mozilla.org/en-US/kb/profiles-where-firefox-stores-user-data#w_finding-your-profile-without-opening-firefox