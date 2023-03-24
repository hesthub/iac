# The ABCs of Ansible

## Intro
Who doesn't like a nice, clean, workspace? Just like how a tidy physical desktop, free from old coffee cups, cable-nests, dishes and candy wrappers can make you feel more productive and focused, a clean and organized development environment can do the same.

But the digital battle of keeping a minimalistic, tidy setup can be tiring, especially if you, like me, love to poke around and prod with new tools and apps on an almost daily basis. 

To have as little as possible, but still all the tools you need for your day-to-day work. 
but how to keep the inevitable bloat under control? 

Do you constantly hunt down temp-folders, config-files, and orphaned libraries for every new application you try out and uninstall? 

How about a fresh install once in a while, hoping to remember which settings to update, what knobs to dial in and which unused features to turn off? 

The savior of our woes might be called Anisble.


### What is Ansible
The solution we are looking into today is Ansible, an IT automation tool.
Written and maintained by Red Hat, its claim to fame is more focused on server provisioning and application deployment, but what is your local environment if not a smaller, more compact infrastructure?

Our goal is to create an automated, version controlled, idempotent configuration that will take us from a freshly installed OS X to a ready-to-go development environment.

Lets dig in.

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
 	ansible.builtin.debug:
  	msg: Hello world
```

and in the same folder as this file we create another inventory: 
>inventory
```
localhost ansible_connection=local
```
This inventory file EXPLAIN

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


so far so good, so lets add a couple of tasks that actually automate our setup. 

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
this task is used to download a dotfile manager called [chezmoi](https://www.chezmoi.io/)
we then move it to path in order to use its CLI to initialize our system from a github repo containing dotfiles. 

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
now this variable will be available for all of our tasks in this playbook. 


## the task
walk through code examples of a simple ansible task or two

## the config
quick overview of the surrounding config used

## the end
conclusion and cliffhanger for part 2 ( roles, requirements and galaxy )`