
# introduction

This post is part two of dipping our toe into the ocean of Ansible.
 
In part 1 of this series, we reviewed the basics of Ansible, how to build and execute tasks, and how to create our first playbook to contain it all. If you missed it, you can find part one here (LINK)

Today we will dive deeper into how we can organize our automation tasks using roles; 
How are roles structured, how can we use them, and how can we customize our tools?

# The structure of a role

Ansible roles allow us to organize and manage tasks found in our playbook.

We can compare roles to containers that group up and hold tasks, variables, files, and customization related to automating a specific part of our setup.

For Ansible to understand our role, we must provide a specific directory structure with subfolders for tasks, handlers, templates, files, variables, and meta, each with a particular purpose: 

```

roles/
    my-role/               # This hierarchy represents a "role."
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
            main.yml      #  <-- role dependencies (left for part 3)

        library/          # roles can also include custom modules
        lookup_plugins/   # or other types of plugins, like lookup in this case

```
Overall, the role structure provides a uniform and standardized way to organize the components we use to build our roles. It makes it easier to manage and gives us a system to follow when cooperating with other developers.

## The tasks 

Let's begin with what we already know: tasks. 

Instead of having all our tasks in a big pile at our playbooks root level, we can segment and collect them into more bite-sized chunks using roles. 

In these roles, we can define our tasks that belong together and combine them with any role-specific configuration and files we need to perform our tasks. Each role in our playbook will work independently of each other and will execute its own set of tasks, which gives us a more modular and flexible approach. 

Sometimes we require quite a few tasks to get the job done, and one approach is to divide them into multiple task files and later merge them at runtime. This separation makes it easier to maintain and modify our playbook over time. 

Let's create a new `Yabai` role that will install and configure a tiling window manager called Yabai (LINK)
`
We'll start by creating a new folder for our role and a subfolder for our tasks:


> mkdir -p yabai/tasks

We create two YAML files inside our tasks folder, one for each task we wish to perform.

> roles/yabai/tasks/yabai.yml

``` yaml
---
- name: "[yabai]: verify yabai is installed"
  community.general.homebrew:
    name: yabai
    state: present
  register: yabai_installed
  until: yabai_installed is succeeded
  Tags:
    - yabai

- name: "[yabai]: configure scripting addition"
  ansible.builtin.lineinfile:
    dest: /private/etc/sudoers.d/yabai
    line: "{{ ansible_user_id }} ALL = (root) NOPASSWD: /usr/local/bin/yabai --load-sa"
    validate: "/usr/sbin/visudo -cf %s."
    create: true
  become: true
  Tags:
    - yabai

- name: "[yabai]: start brew service yabai"
  command: brew services start yabai
  changed_when: false
  Tags:
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
  Tags:
    - yabai

- name: "[yabai]: start brew service skhd"
  command: brew services start skhd
  changed_when: false
  Tags:
    - yabai

```

We can then include these two files in our main tasks file, which serves as the entry point for our role:

> roles/yabai/tasks/main.yml

``` yaml
---

- name: "Install and configure yabai."
  include_tasks: "yabai.yml"

- name: "Install and configure skid."
  include_tasks: "skhd.yml"

```

Finally, once the role is in place and ready to go, we can include it in our play by modifying our init-osx.yml file: 


> init-osx.yml

``` yaml
---
- name: Bootstrap OSX dev machine
  hosts: all

  roles:
    - role: yabai

```


## The vars and the defaults

With the task setup, let's look at how we can handle variables in a role.

In part one, we stored all of our variables inside a global file called config-defaults.yml

Here we have two options, `defaults` and `vars`. 

Both are sets of variables accessible in the same way inside our role, with differences in how they handle precedences and how easy it is to overwrite, with vars being the more rigid and taking precedence.

For a more in-depth look at how Ansible handles its many forms of variables, feel free to read through the official documentation: (LINK)

There are many opinions on splitting our values between defaults and vars, but today we treat them like variables and constants. 

In this case, we treat the `vars` set of variables as more like constants related to the role and the `defaults` set as more dynamic variables that Ansible can overwrite from the playbook case-by-case.

For example, we are looking at a role that handles Homebrew (Link) installation and configuration, a popular package manager for macros. In it, we will be using both defaults and vars.

In the `vars` folder, we have defined values that can be constants for the homebrew role. These values include the expected folder structure for Homebrew and the URL to the Homebrew repository on GitHub. These values are less likely to change in the future, so the `vars` are used as constants.


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
  - Share
  - share/zsh
  - share/zsh/site-functions
  - var
```


On the other hand, the `defaults` folder contains values we want to change on a case-by-case basis. These values include the installation path, any prefixes we wish to use, and a flag determining whether Homebrew should clear the cache after installation.

> roles/homebrew/defaults/main.yml


``` yaml
---


homebrew_prefix: "/usr/local"
homebrew_install_path: "{{ homebrew_prefix }}/Homebrew"
homebrew_brew_bin_path: "{{ homebrew_prefix }}/bin"
homebrew_clear_cache: true

```

We can access our variables wherever we find them, as we saw in the last post, which we can see in our homebrew task here. 

> roles/homebrew/tasks/main.yml

   ``` yaml
   ---
- name: Ensure the Homebrew parent directory has correct permissions.
  file:
    path: "{{ homebrew_prefix }}"
    owner: root
    state: directory
  become: true

- name: Ensure the Homebrew directory exists.
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

# Place brew binary in the proper location and complete setup.
- name: Check if the homebrew binary is already in place.
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
  # the `ansible_user_id` doesn't match the `ansible_user_id.`
  become: false
  become_user: "{{ ansible_user_id }}"

   ``` 
In addition to the variables, we can see a few new Ansible features in this task.

 One of them is the built-in Ansible module `file`.(LINK), which Ansible uses to create, copy, or delete files and set access rights for them


Another feature used is an Ansible plugin called `become` (LINK).
Become is a plugin that ensures that Ansible can use the machine-specific privilege escalation when running commands, which comes in handy for tasks that require writing permission in our user bin folder, for example. 

## The handlers 

Next up, we take a look at handlers, which is a special kind of task that Ansible will only execute when notified by another task.
Ansible often uses handlers to restart an updated service or reload configuration after a task changes the system. 

Let's say we have a role that sets our `localHostName`; The tasks might look something like this: 

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

In the above example, after the task changes the system by setting the local hostname, it notifies a `Clear dns cache` handler to run. Now let's create that handler.

We define our handlers in a separate file within the `handlers` folder of our role, and the name of our task is also the name we use to notify when we want a handler triggered`.

Create a folder inside our new role called handlers and a new YAML file:

> roles/names/handlers/main.yml

``` yaml
---
- name: "Clear dns cache"
  command: "dscacheutil -flushcache"

```

Using handlers in our playbook allows us to create a more explicit workflow, ensuring we only execute tasks when necessary and when all prerequisites for the handler have passed.

Handlers can also be reused for multiple tasks, allowing us to build our playbooks in a more DRY approach.

## The files and the templates

Now that we have our tasks, handlers, and variables set up, let's add some files into the mix.

Ansible manages files using the `copy` and `template` modules. For simple files that don't need any host-dependent data, we can move the files as they are using the `copy` module. But the' template' module shines when it comes to dynamic files.
This module lets us create a new file based on an existing template that contains variables, loops, conditional logic, and more.
We can use this to apply the dynamic configuration to our setup, so if we start to run into more complex infrastructure or if we are using one playbook for multiple machines, templating the config is a great way to reuse basic config and sprinkle customizations on top, and allows for separation of concern regarding config logic and config data.

The `template` module uses the Jinja2 templating engine, a general templating engine for Python. (LINK)

Here's a quick example of how a template might look. Consider a `zshrc` file, and let's template it.

> roles/terminal/templates/zsh.j2


``` bash

export PATH="$HOME/bin:$PATH"

export EDITOR="vim"

alias ls='ls --color=auto'
alias ll='ls -lah'
alias grep='grep --color=auto'

function ssh-agent-start {
  eval $(ssh-agent -s)
  ssh-add
}

# Load any additional scripts using Jinja
{% if additional_scripts %}
{% for script in additional_scripts %}
source {{ script }}
{% endfor %}
{% endif %}

```


The benefits of templating here are that we can reuse a set of standard configuration options for different machines. During runtime, we can apply the customization we need; in this case, we add additional scripts for our system. 

We could have one set of the scripts we use for backend development and a separate one for frontend work; Linux and macOS share the same base configuration, and we apply the difference using templates. 

Once our template file is in place, we can generate our file using the template module inside a task.


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

The template module reads the template file and replaces any placeholders with the corresponding values from our Ansible configuration. This way, we can generate a dynamic configuration file tailored to this specific machine or environment.

``` bash

export PATH="$HOME/bin:$PATH"

export EDITOR="vim"

alias ls='ls --color=auto'
alias ll='ls -lah'
alias grep='grep --color=auto'

function ssh-agent-start {
  eval $(ssh-agent -s)
  ssh-add
}

# Load any additional scripts using Jinja
source /Users/my-user/scripts/macros/script1.sh
source /Users/my-user/scripts/macros/script2.sh

```

## The custom

So far, all the tools and concepts we have discussed have been part of the Ansibles core library. 
But what if we need more than what Ansible currently offers? 
We can create our modules and plugins with Python, some documentation, and an editor. 

We will delve only a little into custom development for Ansible, but the official documentation provides further information.


With modules, we can write small scripts that Ansible can interact with through its API. 

An example would be to create a new Ansible module to interact with the dotfile manager `chezmoi` (LINK),
which we use to download and apply dotfiles to our system. Instead of executing a hardcoded command via the `shell` or `command` modules, we could have a more flexible and direct approach if we develop an interface between Ansible and chezmoi.

Plugins, on the other hand, let us extend and add features to the Ansibles toolkit. 

We have already seen some examples, such as the `become` plugin temporarily elevating system privileges. However, Ansibles uses a wide variety of plugins to handle things like filtering, callbacks, caching, and inventory handling, to name a few. 

For more information on customizing the Ansible toolkit, please refer to Ansible's official documentation: 

- modules (LINK)

- Plugins (LINK)


# Using Roles

Once we have set up our roles with tasks, variables, handlers, files, and templates, we can use them in our playbooks. There are three ways to include roles in a playbook:

- Using the `roles` option at the play level is the classic approach.
- Using the `import_role` at the task level to statically import roles, which behaves like the last option.
- Using the `include_role` at the task level to dynamically reuse a role across the playbook tasks.

## Static usage (classic)

Let's first look at using roles in the classic approach by specifying the roles in the playbook file.

> init-osx.yml

``` yaml
---
- name: Bootstrap OSX dev machine
  hosts: all

  roles:
    - role: names
    - role: dotfiles
    - role: fonts
    - role: Homebrew
    - role: terminal
    - role: yabai
```

The roles we have listed here will be statically imported and processed, and then Ansible will execute the playbook in the following order:

- Any playbook task marked as `pre_task` and their triggered handlers
- Each role in order and execute any dependencies within the role first.
- Any playbook tasks and their triggered handlers
- Any task marked as `post_task` and their handlers

It's worth noting that pre- and post-tasks are a subset of tasks in the playbooks task section that we mark to be executed before or after any other task, whether from a role or role-dependency, third-party, or otherwise.

### tagging

Once we have lined up the roles we want to use in our playbook, we can do some extra housekeeping by tagging our roles and playbook tasks. Depending on their labels, tags will allow us to group and run specific roles and tasks.

For example, if we want to use the same playbook for multiple dev machines, but they all have a core setup in common, we can tag all the roles we consider essential:

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
    - tags: ["ui"]
    - role: Homebrew
    - role: terminal
    - role: yabai
```

We can then selectively run only the roles tagged with `core` using the `--tags` option:

> .ansible-playbook init-osx.yml --tags core

On the other hand, if the core roles are already applied, we can exclude them using the `--skip-tags` option:

> .ansible-playbook init-osx.yml --skip-tags core 

Tagging also applies to individual tasks in a playbook, allowing us to run or skip specific tasks based on their tags. Tags make it possible to fine-tune the playbook's execution depending on the use case.

### passing variables

We can pass parameters to roles from the playbook file by overriding default values provided by the role; for instance, we can override the value of mh_hostname for the role of the name:

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
    - role: Homebrew
    - role: terminal
    - role: yabai
```


Passing variables into a role also allows us to run the role multiple times within the same playbook, which is impossible if we include the role repeatedly. Here's an example that shows a playbook running two instances of a web server, each with different variables:


``` yaml
---
- name: Start critical web servers 
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

## Static usage (tasks)

We can also import roles statically inside the playbooks task section with the `import_role` module. All the same, rules apply but are managed on a task level instead.

`
``` yaml
---
- name: Start critical web servers 
  hosts: all

  tasks:
    - name: Print a message
      ansible.builtin.debug:
        msg: "This task runs our imported roles."

    - name: Import the webserver role statically
      import_role:
        name: webserver
      vars:
        port: 8080
        config: "path/to/config1.cfg"
      tags: ServerA
    ...
```

## Dynamic Usage

Roles can also be dynamically included among our tasks using the `include_role` module. This module allows us to conditionally use roles based on runtime information, making our playbooks more adaptable to the environment and requirements.

One advantage of dynamic roles is their flexibility, allowing us to conditionally import roles based on Ansible facts and `when` modules. For example, we can include the window manager role only on macOS machines:

``` yaml
---
- name: setup macOS`
  hosts: all

  tasks:
    - name: include window manager role based on OS 
      import_role:
        name: yabai
      when: "ansible_facts['os_family'] == 'Darwin'"
    ...
```

Alternatively, we can include roles based on provided variables:

``` yaml
---
- name: setup macOS 
  hosts: all

  tasks:
    - name: include window manager role from variables 
      import_role:
        name: "{{ window_manager_role }}" 
    ...
```

Dynamic roles can simplify complex setups and save processing time, as skipped roles do not need to be handled by Ansible. This feature is handy for multi-machine automation, where the environment and requirements can be large, complex, and ever-changing.

# conclusion

And that wraps up part two of this Ansible series.

We have now gained a more robust and solid understanding of using roles in Ansible to structure our playbooks, automate multiple machines from the same book, and separate complex setups into bite-sized chunks. 

We explored different ways of using roles, including both static and dynamic inclusions. We saw how Ansible uses roles to make our playbooks more maintainable and efficient.

In the next part of this Ansible journey, we will finalize our bootstrap playbook, create our collection, and explore Ansible Galaxy, the hub and repository for sharing Ansible content with other creators; stay tuned.