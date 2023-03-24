# The ABCs of Ansible

## Intro
Who doesn't like a nice, clean, workspace? Just like how a tidy physical desktop, free from old coffee cups, cable-nests, dishes and candy wrappers can make you feel more productive and focused, a clean and organized development environment can do the same.

But the digital battle of keeping a minimalistic, tidy setup can be tiring, especially if you, like me, love to poke around and prod with new tools and apps on an almost daily basis. 

Are you haunted by the flood of temp-folders, config-files, and orphaned libraries, steadily rising for every new application you try out and uninstall? 

A clean slate could solve this, given you remember which settings to update, what knobs to dial in and which unused features to turn off? but what if you dont?

The savior of your woes might be called Anisble.

### What is Ansible

Warning: Ansible is setup, written and configured using YAML, if this causes you alarm, please close your eyes until you've finished reading this post. Moving on.

The solution we are looking into today is Ansible, an IT automation tool.
Written and maintained by Red Hat, its claim to fame is more focused on server provisioning and application deployment, but what is your local environment if not a smaller, more compact infrastructure?

Our goal is to create an automated, version controlled, idempotent configuration that will take us from a freshly installed OS X to a ready-to-go development environment. but we dont have to limit ourselves to a single machine, any machine you can SSH into can be managed this way, from a single controller node. 

Lets dig in and take a look.

### Ansible concepts
Playbooks are the main body of work within Ansible, a collection of all automated actions (tasks) and their configuration we want to perform. 

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
This file describes a playbook with a single task: "print message", which uses an module builtin into Ansible, that will print out our message to the terminal.

We have also specified an attribute
> hosts:all 
which tells anisble to execute our command on all machines (hosts) we have specified in the next file we create, namely

>inventory

```
localhost ansible_connection=local
```

This inventory file manages the collection of hosts we want to apply, here we can group and organize our hosts, manage networking, users, groups, environment variables and more. 
But since this will only cover a single setup for a local machine, lets keep it simple and just add our localhost and call it a day here.

Once we have our hosts set, and task automated we can run this playbook with the following command: 

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
  changed_when: false

- name: Move to PATH
  command: "mv ./bin/chezmoi /usr/local/bin/chezmoi"
  changed_when: false

- name: Initialize dotfiles repo
  command: "chezmoi init --apply {{ github_name }}"
  changed_when: false

```

This task is used to download a dotfile manager called [chezmoi](https://www.chezmoi.io/)
we then move it to path in order to use its CLI to initialize our system from a github repo containing dotfiles. 

TODO CHANGE WHEN!

TODO BECOME FALSE? 

here we also have our first usage of variables in Anisble, instead of hardcoding the github repo we want to init from, we can make our module infer that value from an external source. 

Basic variables such as the string we are using here are denoted by a variable name surrounded by 2 sets of braces like so :

> {{ variable }}

But where do we get our variable-values from? 

in a simple case such as this, we can simple create a file containing our default-values in the root of our playbook.

>default.config.yml

```
---
github_name: your-github-username

```
now this value will be available for all of our tasks in this playbook, and unless overwritten, will be used in place of the variable.

lets add one more task and look at some other ways of working with variables

create a file as follows: 

>fonts.yml

```
---
- name: "[Fonts]: Check if fonts are already installed"
  command: ls /Users/{{ ansible_user_id }}/Library/Fonts/
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
    dest: "/Users/{{ ansible_user_id }}/Library/Fonts/{{ item.1 }}"
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

The first thing to note is the use of a built-in variable called {{ ansible_user_id }}
Available by default and doesn't need to be manually initialized, it points to the current user performing the operation. 
Since we are only running this on our local machine, this is the our own user and can be freely used. 

An ls commands is issued inside our fonts folder, and the result registered inside a local variable, this can then be used to keep the setup idempotent. 

This current_fonts variable is checked for each of our modules using the "when" conditional statement. 
If false, meaning we already have font files in this folder, we skip the execution and move on too the next module in our task.

If applicable (no values set before), we move on and generate our own facts.
the facts we set using the "set_facts" module is a font object containing a list of 2 items. 
These items have a couple of attributes: 
- a name
- an archive/uri
- a directory name
- a list of ttf font files

Using a "loop" module, we iterate over our font objects, creating a temporary folder and downloading and unarchiving the fonts into seperate folders. 

once downloade we use a new, nested loop to iterate over our files attribute inside each font item.

creating a nested loops looks like: 
>{{ fonts | subelements('files') }} 

and once we have seperated our data into an outer and inner loop we access our variables.

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

Before we end, lets take a quick look at some config we can apply to customize our automation experience. 
First of is ansible linting, which can be handled via an .ansible-lint file

>.ansible-lint

```yaml
---
skip_list:
  - experimental
  - fqcn-builtins
```
A list of rule we dont want the linter to warn about.

For more extensive configuration, there is the option to generate an inactive config file using the following command: 

>ansible-config init --disabled > ansible.cfg

this file can then be used as a starting point for any configuration you could want for the playbook.


## the end
conclusion and cliffhanger for part 2 ( roles, requirements,plugins and galaxy )`