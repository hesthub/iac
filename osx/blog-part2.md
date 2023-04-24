This post is part 2 of dipping our toe into the Ansible ocean. 
Today we will dive deeper into how we can orginaze our task using roles; 
How are roles structured, how can we use them and how can we share our roles with other Ansible users. 

## The strucuture of a role

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
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case

### the tasks
    yabai
    https://github.com/ctorgalson/ansible-role-macos-hostname
    https://github.com/markosamuli/ansible-asdf/tree/master/tasks

### the vars and the defaults
    homebrew

### the handlers
    homebrew

### the files and the templates
    terminal / add base thing for iterm2

### the meta
    yabai

### the custom
    library/          # roles can also include custom modules
    module_utils/     # roles can also include custom module_utils
    lookup_plugins/   # or other types of plugins, like lookup in this case

### Using roles

### sharing roles
    Ansible Galaxy - jetbrains
    Collections
    Requirements