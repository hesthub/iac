roles: 

setup window ddmanager - bspwm? sway + wayland
font-fira-code


git clone https://github.com/hest-hub/iac.git

TASK [terminal : Update the paths to the kitty and its icon in the kitty.desktop file(s)] ************************************************************************************************
failed: [localhost] (item=/home/parallels/.local/share/applications/kitty.desktop) => {"ansible_loop_var": "item", "changed": false, "item": "/home/parallels/.local/share/applications/kitty.desktop", "msg": "Unsupported parameters for (ansible.builtin.file) module: content Supported parameters include: _diff_peek, _original_basename, access_time, access_time_format, attributes, follow, force, group, mode, modification_time, modification_time_format, owner, path, recurse, selevel, serole, setype, seuser, src, state, unsafe_writes"}
failed: [localhost] (item=/home/parallels/.local/share/applications/kitty-open.desktop) => {"ansible_loop_var": "item", "changed": false, "item": "/home/parallels/.local/share/applications/kitty-open.desktop", "msg": "Unsupported parameters for (ansible.builtin.file) module: content Supported parameters include: _diff_peek, _original_basename, access_time, access_time_format, attributes, follow, force, group, mode, modification_time, modification_time_format, owner, path, recurse, selevel, serole, setype, seuser, src, state, unsafe_writes"}

