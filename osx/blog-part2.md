This post is part 2 of dipping our toe into ocean of Ansible. 
Today we will dive deeper into how we can orginaze our task using roles; 
How are roles structured, how can we use them and how can we share our roles with other Ansible users. 

### The strucuture of a role

roles/
    my-role/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies

        library/          # roles can also include custom modules
        lookup_plugins/   # or other types of plugins, like lookup in this case

### the tasks

Lets begin with what we already know: tasks. 

Instead of having all of our tasks in a big pile at our playbooks root level, we can instead segment and collect them into more bite sized chunks called roles. 

In our roles we can define our tasks that belong together and combine them with any role specific configuration and files we need to perform our tasks. 

Each role in our playbook will work independent of eachother and will execute their own set of tasks, which gives us a more modular and flexible approach. 

When it comes to tasks themself, sometimes we require quite a few tasks to get the job done, and a good idea is to divide them up into multiple task files, and later merge them togheter at runtime.

Lets create a new role called "yabai" that will install and configure a tiling window manager called Yabai https://github.com/koekeishiya/yabai

We begin with our folder structure


> mkdir -p yabai/tasks

and inside our tasks folder we create two files

> roles/yabai/tasks/yabai.yml

``` yaml
---
- name: "[yabai]: verify yabai is installed"
  community.general.homebrew:
    name: yabai
    state: present
  register: yabai_installed
  until: yabai_installed is succeeded
  tags:
    - yabai

- name: "[yabai]: configure scripting addition"
  ansible.builtin.lineinfile:
    dest: /private/etc/sudoers.d/yabai
    line: "{{ ansible_user_id }} ALL = (root) NOPASSWD: /usr/local/bin/yabai --load-sa"
    validate: "/usr/sbin/visudo -cf %s"
    create: true
  become: true
  tags:
    - yabai

- name: "[yabai]: start brew service yabai"
  command: brew services start yabai
  changed_when: false
  tags:
    - yabai

```

> roles/yabai/tasks/skhd.yml


``` yaml
---
- name: "[yabai]: verify skhd is installed"
  community.general.homebrew:
    name: skhd
    state: present
  register: skhd_installed
  until: skhd_installed is succeeded
  tags:
    - yabai

- name: "[yabai]: start brew service skhd"
  command: brew services start skhd
  changed_when: false
  tags:
    - yabai

```

These two files can then be included in our main file, which is our point of entry to our role. 

In this case we keep it simple and include both our seperated task files, but combining includes with conditionals is an exelect way to control how you roles are used. 

> roles/yabai/tasks/main.yml

``` yaml
---

- name: "Install and configure yabai."
  include_tasks: "yabai.yml"

- name: "Install and configure skhd."
  include_tasks: "skhd.yml"

```

Once the role is in place and ready to go, we can include it in our play by modifying our init-osx.yml file: 


> init-osx.yml

``` yaml
---
- name: Bootstrap OSX dev machine
  hosts: all

  roles:
    - role: yabai

```

### the vars and the defaults

With the task setup, lets take a look at how we can handle variables in a role. 
Here we have two options, "defaults" and "vars" 

Both sets of variables are accessable in the same way, with some differences in precendence and ease to overwrite, with vars being the more rigid and the one that takes precedence. 

for more in depth look at how ansible handles its many forms of variables, feel free to read through the official documentation: 

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html

but for this case we are using the diffrent sets more akin to "vars" acting more like constants related to the role, and defaults as the more dynamic, case by case variables we overwrite on demand. 


for this role we have setup both defaults and vars.

in our more "static" variables we have set the excpected homebrew folder structure and a url to homebrews github repository, which probably wont change anytime soon. 

> roles/homebrew/vars/main.yml

``` yaml
---

homebrew_repo: https://github.com/Homebrew/brew

homebrew_folders_base:
  - Cellar
  - Homebrew
  - Frameworks
  - Caskroom
  - bin
  - etc
  - include
  - lib
  - opt
  - sbin
  - share
  - share/zsh
  - share/zsh/site-functions
  - var
```

In our defaults we set some usable prefixes, desired paths and conditional flags.
These variables are a prime candidate to override dependeing if you are running OSX on an Intel or Arm machine.

> roles/homebrew/defaults/main.yml


``` yaml
---


homebrew_prefix: "/usr/local"
homebrew_install_path: "{{ homebrew_prefix }}/Homebrew"
homebrew_brew_bin_path: "{{ homebrew_prefix }}/bin"
homebrew_clear_cache: true

```

Whichever we found our variables, we can access them the same way as we saw in the last post, which we can see in our homebrew task here. 

Besides variables, we can also see a couple of new features in this task.

Firstly we can see heavy use the built-in module file https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html

we also see the use of an ansible plugin called "become" https://docs.ansible.com/ansible/latest/plugins/become.html
which works to ensure that Ansible can use privilege escalation when running commands. 

> roles/homebrew/tasks/main.yml

   ``` yaml
   ---
- name: Ensure Homebrew parent directory has correct permissions.
  file:
    path: "{{ homebrew_prefix }}"
    owner: root
    state: directory
  become: true

- name: Ensure Homebrew directory exists.
  file:
    path: "{{ homebrew_install_path }}"
    owner: "{{ ansible_user_id }}"
    group: "{{ ansible_user_gid }}"
    state: directory
    mode: 0775
  become: true

# Clone Homebrew.
- name: Ensure Homebrew is installed.
  git:
    repo: "{{ homebrew_repo }}"
    version: master
    dest: "{{ homebrew_install_path }}"
    update: false
    depth: 1
  become: true
  become_user: "{{ ansible_user_id }}"

# Adjust Homebrew permissions.
- name: Ensure proper permissions and ownership on homebrew_brew_bin_path dirs.
  file:
    path: "{{ homebrew_brew_bin_path }}"
    state: directory
    owner: "{{ ansible_user_id }}"
    group: "{{ ansible_user_gid }}"
    mode: 0775
  become: true

- name: Ensure proper ownership on homebrew_install_path subdirs.
  file:
    path: "{{ homebrew_install_path }}"
    state: directory
    owner: "{{ ansible_user_id }}"
    group: "{{ ansible_user_gid }}"
    recurse: true
  become: true

# Place brew binary in proper location and complete setup.
- name: Check if homebrew binary is already in place.
  stat: "path={{ homebrew_brew_bin_path }}/brew"
  register: homebrew_binary
  check_mode: false

- name: Symlink brew to homebrew_brew_bin_path.
  file:
    src: "{{ homebrew_install_path }}/bin/brew"
    dest: "{{ homebrew_brew_bin_path }}/brew"
    state: link
  when: not homebrew_binary.stat.exists
  become: true

- name: Ensure proper homebrew folders are in place.
  file:
    path: "{{ homebrew_prefix }}/{{ item }}"
    state: directory
    owner: "{{ ansible_user_id }}"
    group: "{{ ansible_user_gid }}"
  become: true
  loop: "{{ homebrew_folders_base }}"

- name: Collect package manager fact.
  setup:
    filter: ansible_pkg_mgr

- name: Perform brew installation.
  block:
    - name: Force update brew after installation.
      command: "{{ homebrew_brew_bin_path }}/brew update --force"
      when: not homebrew_binary.stat.exists

    - name: Where is the cache?
      command: "{{ homebrew_brew_bin_path }}/brew --cache"
      register: homebrew_cache_path
      changed_when: false
      check_mode: false

    - name: Check for Brewfile.
      stat:
        path: "{{ lookup('env', 'HOME') }}/Brewfile"
      register: homebrew_brewfile
      check_mode: false

    - name: Install from Brewfile.
      command: "{{ homebrew_brew_bin_path }}/brew bundle chdir={{ lookup('env', 'HOME') }}"
      when: homebrew_brewfile.stat.exists

    - name: Upgrade all homebrew packages (if configured).
      community.general.homebrew: update_homebrew=yes upgrade_all=yes
      notify:
        - Clear homebrew cache

  # Privilege escalation is only required for inner steps when
  # the `ansible_user_id` doesn't match the `ansible_user_id`
  become: false
  become_user: "{{ ansible_user_id }}"

   ``` 

### the handlers

Next up we take a look at handlers, which are tasks that are only executed when notified by another task. 
Handlers are a often used to restart an updated service, or reload configuration after a task has made changes to the system. 

Lets say we have a role that sets our hostName,localHostName and ComputerName. 

one of the tasks might look something like this: 


> roles/names/localhostname.yml

``` yaml 
---
- name: Find out the current macOS LocalHostName value.
  command: "scutil --get LocalHostName"
  register: "mh_current_localhostname"
  changed_when: false
  failed_when: false

- name: Set macOS LocalHostName.
  command: "scutil --set LocalHostName '{{ mh_localhostname }}'"
  when:
    - "mh_localhostname != mh_current_localhostname.stdout"
  become: true
  notify: Clear dns cache

```

here we fetch the current value, register it to a temporary variable and compares it to our mh_localhostname variable found in a defautls folder. 

if these values differ, we use the command module and set our LocalHostName, and if everything goes according to plan, we use the "notify" keyword to call a task named "Clean dns cache".

This will call the handler we have setup in our role structure: 

> roles/names/handlers/main.yml

``` yaml
---
- name: "Clear dns cache"
  command: "dscacheutil -flushcache"

```

So when we successfully set our new localHostName, we trigger a dns cache flush, making sure we dont run into any strange behaviours with old cached names. 

Using handlers created a flow that ensures that dependent tasks are executed only when necessary, and only once all pre-requisitets are completed, making our playbooks more efficent and reliable. 

We can also reuse the same handlers for mutiple tasks, allowing for a more DRY approach to our playbooks.

### the files and the templates

In Ansible, files are manged using the "copy" and "template" modules. for simple files that doesn't need any host-dependant data, we use copy to simply copy our files onto our system. 

The "template" on the other hand, lets us create a new file based on a existing template, which can contain variables, loops, conditional logic and more.  

This file is written in an templating engine called jinja2 https://github.com/pallets/jinja/

for example, we might want to template our zsh config file, instead of copying it from a repository. this might look something like this: 

> roles/terminal/templates/zsh.j2


``` bash

# .zshrc template
# This file is generated by Ansible, do not modify manually.

# Set the PATH variable
export PATH="$HOME/bin:$PATH"

# Set the default editor
export EDITOR="vim"

# Aliases
alias ls='ls --color=auto'
alias ll='ls -lah'
alias grep='grep --color=auto'

# Functions
function ssh-agent-start {
  eval $(ssh-agent -s)
  ssh-add
}

# Load any additional scripts
{% if additional_scripts %}
{% for script in additional_scripts %}
source {{ script }}
{% endfor %}
{% endif %}


```

Here we have some base, common configuration for our zsh setup, we setup our path, default editor and a couple of aliases and scripts, but then we are using our template language to inject additional script, based on our ansible configuration. 

Maybe we have one set of script we want to use when setting up a envrionment for backend development, and a seperate one for frontend work, maybe linux and macOS share the same base configuration, and we apply the diffrance using templates. 


> roles/terminal/tasks/main.yml

``` yaml 
- name: Configure .zshrc
  template:
    src: zshrc.j2
    dest: /Users/{{ansible_user_id}}/.zshrc
  vars:
    additional_scripts:
      - /Users/{{ansible_user_id}}/scripts/macos/script1.sh
      - /Users/{{ansible_user_id}}/scripts/macos/script2.sh

``` 

Templating this file will generate a new .zshrc file and insert our scripts into it based on our configuration and would look something like this: 

``` bash

# .zshrc template
# This file is generated by Ansible, do not modify manually.

# Set the PATH variable
export PATH="$HOME/bin:$PATH"

# Set the default editor
export EDITOR="vim"

# Aliases
alias ls='ls --color=auto'
alias ll='ls -lah'
alias grep='grep --color=auto'

# Functions
function ssh-agent-start {
  eval $(ssh-agent -s)
  ssh-add
}

# Set the PATH variabl# Load any additional scripts
source /Users/my-user/scripts/macos/script1.sh
source /Users/my-user/scripts/macos/script2.sh

```


### the meta
the meta of our role contains, unsuprisingly, meta-data about or role. 

Here we can declare information such as:

- author
- role description
- license
- supported platforms
- other roles used as dependecies to our role
- versioning information for both our role, platforms and depenencies

``` yaml
---
galaxy_info:
  role_name: init OSX
  author: Henrik Starefors
  description: bootstrap OSX 
  license: MIT
  min_ansible_version: 2.6
  platforms:
    - name: MacOSX
      versions:
        - '10.13'
        - '10.14'
  galaxy_tags:
    - Development
    - System
    - MacOSX
dependencies: []

```

the meta data of our role is used to describe our role and its interaction with other roles when we share our role with other through the hub called Ansible Galaxy, which we will look into more in just a bit. 

### the custom

So far, all the tools, modules, plugins and templates we have been talking about has been part of Ansibles core library. 
But what if we need more than Anisble currently offers? 
With a little bit of Python, some documentaiton, and an editor we are able to create our very own modules and plugins. 

We won't be digging to far into custom development for Ansible, but more information can be found in the official documentation: 


modules https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html#creating-a-module

With modules we can write small script that Ansible can interact with through its API. 

An example would be to create a new Ansible module to interact with the dotfile manager chezmoi https://www.chezmoi.io/ we use to download and apply dotfiles to our system, so instead of executing a hardcoded command, we could have a more flexible and direct approach if we develop an interface between Ansible and chezmoi.

Plugins https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.html#developing-particular-plugin-types

Plugins on the other hand lets us extend and add features to Ansibles toolkit. 

We have already seen a couple of examples such as the "become" plugin that lets us temprarely take on a more privilages system role, but there is a sleugh of diffrent plugins we can leverage to handle things like filtering, callbacks, caching, inventory handling and much more. 


### Using roles, static import

Now we have our roles, they are structured with tasks, variables, handlers, files and templates, maybe even some custom plugins under the hood. Lets take a look at how we can use them in our playbooks. 

There are three ways we can use our roles: 

- at the play level with the "roles" option: the classic.
- at the task level with "import_role": importing roles at a task level, otherwise the same as the classic option.
- at the task level with "include_role": here we can dynamicly reuse our role as we please.


So lets use our roles the classic way, from the entrypoint of our playbook

> init-osx.yml

``` yaml
---
- name: Bootstrap OSX dev machine
  hosts: all

  roles:
    - role: names
    - role: dotfiles
    - role: fonts
    - role: homebrew
    - role: terminal
    - role: yabai
```

The roles we have listed here will be staticly imported and process and then Ansible will execute the playbook in the following order: 

- Any "pre_task" and triggered handlers
- each role listed, in order, and if any depedencies are found within the role, they are executed before the role itself.
- Any tasks and their triggered handlers
- an "post_task" and their handlers


All of our roles and task can be tagged within a playbook, allowing us to group and run specific tasks depending on their tags. 

Say we want to use the same playbook for multiple dev machines, but they all have a core setup in common, then we can tag all the roles we consider essential: 

``` yaml
---
- name: Bootstrap OSX dev machine
  hosts: all

  roles:
    - role: names
      tags: ["core"]
    - role: dotfiles
      tags: ["core"]
    - role: fonts
    - role: homebrew
    - role: terminal
    - role: yabai
```

we can then selectivy run only the roles tagged with "core": 

> .ansible-playbook init-osx.yml --tags core

or if the core roles have already been applied we can exclude them instead: 

> .ansible-playbook init-osx.yml --skip-tags core 


from here we can also pass in parameters to our roles, for instance if we want to change our default values we can override the value found in our role: 

``` yaml
---
- name: Bootstrap OSX dev machine
  hosts: all

  roles:
    - role: names
      vars:
        mh_hostname: "override-name"
    - role: dotfiles
    - role: fonts
    - role: homebrew
    - role: terminal
    - role: yabai
```

The default behavior for a role that is included multiple times is to only run it once, but passing diffrent vars lets us run the same role mutiple times whitin one playbook. 

The following example shows a playbook running two instances of a webserver, with diffrent vars:

``` yaml
---
- name: Start very important webservers 
  hosts: all

  roles:
    - role: setup
    - role: webserver
      vars:
        port: 8080
        config: "path/to/config1.cfg"
      tags: ServerA
    - role: webserver
      vars:
        port: 8081
        config: "path/to/config2.cfg"
      tags: ServerB
```

If we instead prefer to work at a task level in our playbook, all the same static rules appies to a task importing the roles staticly in our playbook with the "import_role": 

``` yaml
---
- name: Start very important webservers 
  hosts: all

  tasks:
    - name: Print a message
      ansible.builtin.debug:
        msg: "this task runs our imported roles"

    - name: Import the webserver role staticly
      import_role:
        name: webserver
      vars:
        port: 8080
        config: "path/to/config1.cfg"
      tags: ServerA
    ...
```

### including roles dynamicly

Roles can also be dynamicly included in our tasks using the "include_role". 
These roles will be determined at runtime, and runs at the task level of our playbook. 
The main advantage of dynamic roles is the flexibilty it provides, allowing us to conditionally import roles, by using Ansible facts and "when" statements: 

``` yaml
---
- name: setup macos 
  hosts: all

  tasks:
    - name: include window manager role based on OS 
      import_role:
        name: yabai
      when: "ansible_facts['os_family'] == 'Darwin'"
    ...
```

or we can include our roles based on provided variables: 

``` yaml
---
- name: setup macos 
  hosts: all

  tasks:
    - name: include window manager role from variables 
      import_role:
        name: "{{ window_manager_role }}" 
    ...
```


### sharing roles

One last thing before we wrap things up: sharing our Ansible creations. 

https://galaxy.ansible.com/

Ansible Galaxy is a central repository and  hub for community-developed roles.
It allows us to download, share and package roles that has been created, tested and verfied by other Ansible users.

Anyone can publish to Galaxy, sharing their work and spreading knowledge with the community.

Galaxy lets us discover roles and include them in our own playbook, no need to re-invent the wheel for common tasks and automations. 

But its not only roles we can find in the hub, here we can also find collections. 
Ansible collections are turn-key packages that can contain roles, modules, plugins, files and entire playbooks, and provides a strucuted way of distributing Ansible content.

So say we search the hub and find a role we would like to use for example "elliotweiser.osx-command-line-tools".
https://galaxy.ansible.com/elliotweiser/osx-command-line-tools

Now we can manually download it using the galaxy-client: 


> ansible-galaxy install elliotweiser.osx-command-line-tools


or use it as a dependcy to our playbook using a Ansible requirements file 


> requirements.yml


``` yaml
---
roles:
  - name: elliotweiser.osx-command-line-tools

```

this will tell Ansible to download and install this role from Galaxy as part of our playbook. 
once in place, we can use the role as it where our own: 


``` yaml
---
- name: Bootstrap OSX dev machine
  hosts: all

  roles:
    - role: elliotweiser.osx-command-line-tools
    - role: names
    - role: dotfiles
    ...
```


## conclusion

And that wraps up part two of this Ansible series. 
With a bit more role-knowledge under our belt we can now start building our roles, automate multiple machines from the same playbook, and share our findings with the world through Ansible Galaxy. 

In the third and final part of this series we will bring it all together and bootstrap a machine from fresh install into a working, ready to go development enviroment, just the way we like it. 