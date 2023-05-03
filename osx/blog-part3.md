### the meta

And now for one last piece of the Ansible roles core structure: the Meta.

The meta of our role contains, unsuprisingly, meta-data about our role. 

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

the meta data of our role is used to describe our role and its interaction with other roles when we share our role, of pull down other peoples roles through the hub called Ansible Galaxy, which we will look into more in just a bit. 


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