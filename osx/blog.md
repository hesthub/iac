# The ABCs of Ansible

## Intro
Who doesn't like a nice, clean, workspace? Just like how a tidy physical desktop, free from stacks of coffee cups, old dishes and candy wrappers can make you feel more productive and focused, a clean and organized development environment can do the same.

Do you find yourself battling the chaos of temporary folders, configuration files, and orphaned libraries that only seem to multiply with every new application you test and uninstall?

While maintaining a well-organized environment can help you stay in control, it's not always practical, or even possible to remain neat and orderly day-to-day; searching for, tidying up and trying to keep the bloat at bay. 

Then what can we do about it?
Enter Anisble, the savior of our woes, offering a clean slate approach without forcing you to remeber which settings to update, what knobs to dial in and which unused features to turn off every time you reinstall your machine.


### What is Ansible
Caution: Ansible's setup, configuration, and scripting rely on YAML, if this causes you alarm, please close your eyes until you've finished reading this post.

The solution we are looking into today is Ansible, an IT automation tool written and maintained by Red Hat.
Ansibles claim to fame might be more focused on server provisioning and application deployment, but what is your local environment if not a smaller, more compact infrastructure? And just like our ifrastructure, we want to raise cattle, not pets, meaning that idealy we want to be able to replicate our enviroment on any machine with the least amount of friction, without losing our settings, preferences, personal scripts and dotfiles.

To achive this, our objective is to create an automated, version controlled, idempotent configuration that will take us from a freshly installed machine to a ready-to-go development environment with the press of a button, or to be more precise: the execution of a script.

Lets dig in and take a look. We are taking a look at the building blocks of ansible. 

    Inventory: The inventory is a file or a collection of files that defines the hosts (servers, network devices, etc.) that Ansible manages.

    Playbooks: Playbooks are the heart of Ansible. They define the automation tasks, configurations, and orchestration to be performed on the inventory hosts.

    Tasks: Tasks are the smallest unit of work in Ansible. They are a series of actions to be performed on the target hosts, using modules to provide functionalities.

    Modules: Modules are the units of code that Ansible executes. with modules we perform our actions, such as installing packages, managing files, and configuring services. There are hundreds of built-in modules, and you can create custom ones as well.

    Roles: Roles are a way to organize and reuse Ansible content. They encapsulate tasks, variables, templates, files, and handlers in a standardized structure, making it easier to share and reuse code across projects. 

    Templates: Templates are text files that can contain dynamic content, using the Jinja2 templating engine. They allow you to generate configuration files based on variables, making it easy to create configuration based on host and other varibales.

    Handlers: Handlers are special types of tasks that are triggered only when a specific event occurs, such as a change in the configuration of a service. They are commonly used to restart services when their configurations are updated.

    Ansible Galaxy: a community-driven hub and repository for sharing Ansible collections and roles for other to reuse and extend.

In this post we will be focusing on the core components of Ansible: inventory, playbooks, tasks and modules, saving the organizational-focused parts for a later time. 

### Ansible in action
Playbooks are the main body of work within Ansible, and together with the inventory comprises the core of our setup.

to create a new playbook we simply need to create a yaml file with a descriptive name, with the following contents: 

>init-osx.yml 

```yaml
---
- name: Bootstrap OSX dev machine
  hosts: all

  tasks:
   - name: Print message
 	debug:
  	msg: Hello world
```
This file describes a playbook with a single task: "print message", which uses a builtin module, that will print out our message to the terminal.

We have also specified an attribute
> hosts:all 
which tells anisble to execute our command on all machines (hosts) we have specified in our inventory

>inventory

```
localhost ansible_connection=local
```

The inventory manages the collection of hosts we want to apply, and since we are focusing on just our local machine, this inventory will be kept quite simple.

Once we have our hosts set, playbook ready and task automated we can run this with the following command: 

> ansible-playbook -i inventory init-osx.yml
 
 which should give us a response similar to this: 

```css
 PLAY [Bootstrap OSX dev machine] *******************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************
ok: [localhost]

TASK [Print message] *******************************************************************************************************************
ok: [localhost] => {
    "msg": "Hello world"
}

PLAY RECAP *****************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

Here we can see: 

- the name of our play and tasks
- Ansibles inplicit "Gathering facts" task that runs before our own tasks, used to get information from our inventory
- the status of each task
- a summary of all tasks ran and their status.

so far so good, so lets add a couple of tasks that does some actual work to automate our setup. 

first of we create a task folder and our first task inside it 

> tasks/dotfiles.yml

```yaml
---
- name: Download chezmoi
  shell: 'sh -c "$(curl -fsLS get.chezmoi.io)" '

- name: Move to PATH
  command: "mv ./bin/chezmoi /usr/local/bin/chezmoi"

- name: Initialize dotfiles repo
  command: "chezmoi init --apply {{ github_name }}"

```


This task is used to download a dotfile manager called [chezmoi](https://www.chezmoi.io/)
we then move it to path in order to use its CLI to initialize our system from a github repo containing dotfiles. 

this is done in three steps, and as we can see using two diffrent builtin modules: shell and command. 

Both are simillar, used to execute commands on the host, command being the prefered, safer option, but shell having the added benefit of actually execution via a shell, and therefore being able to access enviroment variables, process operations like "|", "&", ">", "<" and more.

Here we also have our first usage of variables in Anisble, instead of hardcoding the github repo we want to init from, we can make our module infer that value from an external source. 

Basic variables such as the string we are using here are denoted by a variable name surrounded by 2 sets of braces like so :

> {{ variable-name }}

But where do we get our variables value from? 

in a simple, gathered up case such as this, we can create a file containing our default-values in the root of our playbook.

>default.config.yml

```yaml
---
github_name: your-github-username

```
now this value will be available for all of our tasks in this playbook, and unless overwritten, will be used in place of the variable.

lets add one more task and look at some other ways of working with variables. 

Create a new task file with the following content: 

>fonts.yml

```yaml
---
- name: "[Fonts]: Check if fonts are already installed"
  command: ls {{ ansible_user_dir }}/Library/Fonts/
  register: current_fonts

- name: "[Fonts]: Set fonts variable"
  set_fact:
    fonts:
      - name: FiraCode
        archive: https://github.com/tonsky/FiraCode/releases/download/6.2/Fira_Code_v6.2.zip
        directory: ttf
        files:
          - FiraCode-Bold.ttf
          - FiraCode-Light.ttf
          - FiraCode-Medium.ttf
          - FiraCode-Regular.ttf
          - FiraCode-Retina.ttf
      - name: Monoid
        archive: https://github.com/JB-Dmitry/monoid/blob/master/Monoid-0.61-with-IntelliJ-support.zip?raw=true
        directory: ""
        files:
          - Monoisome-Regular.ttf
          - Monoid-Retina.ttf
          - Monoid-Bold.ttf
          - Monoid-Italic.ttf
  when: fonts is undefined

- name: "[Fonts]: Create a directory if it does not exist"
  ansible.builtin.file:
    path: "/tmp/fonts-{{ item.name }}"
    state: directory
    mode: "0755"
  loop: "{{ fonts }}"
  when: current_fonts == ""

- name: "[Fonts]: Download and Extract font files"
  unarchive:
    src: "{{ item.archive }}"
    dest: "/tmp/fonts-{{ item.name }}"
    remote_src: true
  loop: "{{ fonts }}"
  when: current_fonts == ""

- name: "[Fonts]: Install fonts from repositories"
  copy:
    src: "/tmp/fonts-{{ item.0.name }}/{{ item.0.directory }}/{{ item.1 }}"
    dest: "{{ ansible_user_dir }}/Library/Fonts/{{ item.1 }}"
  loop: "{{ fonts | subelements('files') }}"
  when: current_fonts == ""

- name: "[Fonts]: Remove repositories"
  file:
    path: "/tmp/fonts-{{ item.name }}"
    state: absent
  changed_when: false
  loop: "{{ fonts }}"
  when: current_fonts == ""

```

The first thing to note is the use of a built-in variable called {{ ansible_user_dir }}.
This variable is set by Ansible during the gathering_facts phase, and globally available inside the playbook. 
If you are intresested in what other facts Ansible has gathered about the system, we can create a small task to print them all with this: 

```yaml
  tasks:
    - name: Print all available facts
      ansible.builtin.debug:
        var: ansible_facts

```

Using the builtin variable to reach our home folder, an ls commands is issued inside our Librabries fonts folder, and the result registered inside a local variable (current_fonts).

The current_fonts variable is then checked for each of our modules using the "when" conditional statement, and is one of the ways we can keep our configuration idempotent.

"When" is used to determine if a specifc module should run or not, for instance, no need to download a new set of fonts if they are already installed. 

If applicable (no values set before), we move on and generate our own facts.
the facts we set using the "set_facts" module is an object containing a list of 2 items. 
These items in term have a couple of attributes: 
- a name
- an archive/uri
- a directory name
- a list of ttf font files

In order to iterate through our obects we make use of the builtin "loop" module, which will treverse over the top layer objects, and return a variable called item that contains the result of our loops, so in order to get the name of our font objects we simply call 
> {{ item.0.name }}

 then we use a second, nested loop to iterate over our files attribute inside each font item.

creating a nested loops looks like: 
>{{ fonts | subelements('files') }} 

and once we have seperated our data into an outer and inner loop we access our variables in the following way.

To get a hold of our variable name in fonts.[1..n].name

>{{ item.0.name }}

and later to iterate through our files name, nested inside fonts.[1..n].files[1..n]

>{{ item.1 }} 

with these modules we now move our fonts from the temporary folder to our library to install them, and later clean up and delete the temporary directories. 

all we need to do now is add this task too our playbook.

```yaml
  tasks:
    - name: Dotfiles
      import_tasks: tasks/dotfiles.yml
      tags: ["dotfiles"]
    - name: Fonts
      import_tasks: tasks/fonts.yml
      tags: ["dotfiles"]
```
Then execute it, same as before 

> ansible-playbook -i inventory init-osx.yml

And with that we have a simple ansible playbook to automaticly fetch and initilaze our dotfiles from github, and install a set of nice fonts we can use. 

Before we end, lets take a quick look at some config we can apply to customize our Ansible experience. 
First of is linting, which can be handled via an .ansible-lint file

>.ansible-lint

```yaml
---
skip_list:
  - experimental
  - fqcn-builtins
```
A list of rule we dont want the linter to warn about.
here we are removing warnings about using "fully qualified collection name", so instead of writing "ansible.builtin.command" we can simply write "command"

For more extensive configuration, there is the option to generate an inactive config file using the following command: 

>ansible-config init --disabled > ansible.cfg

this file can then be used as a starting point for any configuration you could want for the playbook.
We have the option to set configuration such as if and how you want Ansible to gather facts about the host, output format, timeouts how to deal with errors and much more. 

## the end
And with that, we now have a solid starting point for automating our local environment!

Today we've merely scratched the surface, exploring the building blocks and executing basic playbooks. 

In our upcoming post, we'll will use the new knowledge we have gained and delve deeper into the power of roles for creating self-contained sets of tasks, complete with isolated configurations and rulesets. 

We'll also investigate Ansible templates to generate our config files based on host variables and other conditions, as well as tapping into the Ansible community to reuse existing roles and collections, eliminating the need to reinvent the wheel for common, already solved tasks.