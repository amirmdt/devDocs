# Install ansible on ubuntu

*start ubuntu app on windows using" `wsl` *command on powershell or cmd*

`sudo apt update`

*install pip using command* `sudo apt install python3-pip`

*`pip3 --version` to check version of pip installed*

*i tried to install ansible using command `python3 -m pip install --user ansible` i got below error:*

```
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.

    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
    sure you have python3-full installed.

    If you wish to install a non-Debian packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.

    See /usr/share/doc/python3.12/README.venv for more information.

note: If you believe this is a mistake, please contact your Python installation or OS distribution provider. You can override this, at the risk of breaking your Python installation or OS, by passing --break-system-packages.
hint: See PEP 668 for the detailed specification.
```

***This error occurs because Ubuntu (starting from 23.04+) enforces PEP 668, which prevents pip from installing packages system-wide to avoid conflicts with apt-managed Python packages.***

*there are 3 safe ways to install ansible*

1. Use apt (Recommended for System-Wide Installation)\
*Since Ansible is available in Ubuntu's repositories, install it via apt:*
```
sudo apt update
sudo apt install ansible
```
*Check the installed version:*\
```
ansible --version
```

2. Use a Python Virtual Environment (Recommended for Isolated Installation)\
*If you need a newer version of Ansible than what apt provides, use a virtual environment:*

*Install python3-venv (if not installed):*
```
sudo apt install python3-venv
```

*Create and activate a virtual environment:*
```
python3 -m venv ~/ansible-venv
source ~/ansible-venv/bin/activate
```

*Install Ansible inside the virtual environment:*
```
pip install ansible
```

*Verify:*
```
ansible --version
```

*Deactivate the virtual environment when done:*
```
deactivate
```

3. Use `pipx` (Best for CLI Applications like Ansible)
*pipx installs Python apps in isolated environments without affecting the system:*

*Install pipx:*
```
sudo apt install pipx
pipx ensurepath
```

*Install Ansible with pipx:*
```
pipx install ansible
```

*Verify:*
```
ansible --version
```

# ansible ad hoc commands

***before everything you should add your control node sytem (the machine that runs Ansible) ssh pub key to ***authorized_keys*** file in target node***

***remember your target node should has static ip (because it should'nt change)***

*to ping localhost by ansible* ***`ansible localhost -m ping`***

***to ping another host as target node follow below :***

*for example you have a server which its ip is 192.168.12.4 and its user is host1*

*in control node create a file named inventory.ini (you can create inventory.yaml too)*

*add following to file `inventory.ini`:*\
`node001 ansible_host=192.168.12.4 ansible_user=host1`\
*if you want to add another target node the file will be like:*
```
node001 ansible_host=192.168.12.4 ansible_user=host1
node002 ansible_host=192.168.12.5 ansible_user=host2
```

*now you can use command* 
```
ansible node001 -m ping -i inventory.ini
```
*to ping that target node*

***command below will give you hardware and software information about target node***
```
ansible node001 -m gather_facts -i inventory.ini
```

***it is possible system ask you for key cheking like below***

```
The authenticity of host '192.168.12.4 (192.168.12.4)' can't be established.
ED25519 key fingerprint is SHA256:kXrVYuZJW/lBVGyNXEz+TqXaFMBxAeXnZydQfuigMOA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
*it is not going to happen in new versions but if this happened use the following command :*
```
export ANSIBLE_HOST_KEY_CHECKING=false
```
*if you change false with true it will ask again (just first time after first time the files known_hosts will be created in .ssh directory)*

# Write inventory files more systematic

*for example you have 5 target nodes. you can create inventory file like below:*

```
node001 ansible_host=192.168.12.4 ansible_user=host1
node002 ansible_host=192.168.12.5 ansible_user=host2
node003 ansible_host=192.168.12.6 ansible_user=host3
node004 ansible_host=192.168.12.7 ansible_user=host4
node005 ansible_host=192.168.12.8 ansible_user=host5
```
*this is not a smart way to write the file like that*

*to write it cleaner first you can add name and ip of target nodes to `/etc/hosts` file like:*
```
192.168.12.4    node001
192.168.12.5    node002
192.168.12.6    node003
192.168.12.7    node004
192.168.12.8    node005
```
*then you can ignore writing ***ansible_host*** in inventory file. the inventory file will be like :*
```
node001 ansible_user=host1
node002 ansible_user=host2
node003 ansible_user=host3
node004 ansible_user=host4
node005 ansible_user=host5
```

*if your target nodes ansible_users (host1) or any variable of them is the same you can change the file like below :*\
```
[webservers]
node001
node002
node003
node004
node005

[webservers:vars]
ansible_user=host1
```

*and you can ping all webservers nodes using :* ***`ansible webservers -m ping -i inventory.ini`***

*the file can be written even better :*
```
[webservers]
node[001:005]

[webservers:vars]
ansible_user=host1
```

*now write the file with better classification*
```
[apache]
node[001:004]

[nginx]
node005

[webservers:children]
nginx
apache

[webservers:vars]
ansible_user=host1
```

### write above inventories in YAML format

***Examples***

*ini*
```
[webservers]
node001 ansible_host=192.168.12.4 ansible_user=host1
node002 ansible_host=192.168.12.5 ansible_user=host2
node003 ansible_host=192.168.12.6 ansible_user=host3
node004 ansible_host=192.168.12.7 ansible_user=host4
node005 ansible_host=192.168.12.8 ansible_user=host5
```
*yaml*
```
webservers:
  hosts:
    node001:
      ansible_host: 192.168.12.4
      ansible_user: host1
    node002:
      ansible_host: 192.168.12.5
      ansible_user: host2
    node003:
      ansible_host: 192.168.12.6
      ansible_user: host3
    node004:
      ansible_host: 192.168.12.7
      ansible_user: host4
    node005:
      ansible_host: 192.168.12.8
      ansible_user: host5
```
________

*ini*
```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```
*yaml*
```
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
```

### Default groups

***Even if you do not define any groups in your inventory file, Ansible creates two default groups: `all` and `ungrouped` after integrating all inventory sources. The `all` group contains every host. The `ungrouped` group contains all hosts that don't have another group aside from `all`. Every host will always belong to at least 2 groups (`all` and `ungrouped` or `all` and some other group). For example, in the basic inventory above, the host `mail.example.com` belongs to the `all` group and the `ungrouped` group; the host `two.example.com` belongs to the `all` group and the `dbservers` group. Though `all` and `ungrouped` are always present, they can be implicit and not appear in group listings like `group_names` .***

*ini*
```
[apache]
node[001:004]

[nginx]
node005

[webservers:children]
nginx
apache

[webservers:vars]
ansible_user=host1
```
*yaml*
```
all:
  children:
    apache:
      hosts:
        node[001:004]
    nginx:
      hosts:
        node005
    webservers:
      children:
        apache
        nginx
      vars:
        ansible_user: host1
```

## Playbook

*Ansible Playbooks offer a repeatable, reusable, simple configuration management and multi-machine deployment system, one that is well suited to deploying complex applications. If you need to execute a task with Ansible more than once, write a playbook and put it under source control. Then you can use the playbook to push out new configuration or confirm the configuration of remote systems.*

*Playbooks can:*

* declare configurations

* orchestrate steps of any manual ordered process, on multiple sets of machines, in a defined order

* launch tasks synchronously or asynchronously


### first playbook

```
---
- name: my first playbook
  hosts: node001:node002
  tasks:
    - name: brief definition of module
      MODULE_NAME:
        MODULE_PROPERTIES:
```
*a playbook is like what is above. starts with `---` . then name the hosts which this playbook will do its jobs on them. after that using `tasks:` you can use modules to tell playbook what to do.*\
*`- name:` is a brief definition of what module is going to do. `MODULE_NAME:` is the name of module (for example `apt` or `debug`). `MODULE_PROPERTIES:` as it is obvious is properties of that module*

*now if you want to run the playbook you should use command below:*\
`ansible-playbook playbookfile.yml -i inventoryfile.yml`

### debug module

*the `debug` module is used to print messages, variable values, or expressions during playbook execution. it can be written like below in a playbook:*
```
- name: my first playbook
  hosts: node001:node002
  tasks:
    - name: print hello to ansible
      debug:
        msg: "hello ansibler"
```

### fail module

*In Ansible, the `fail` module is used to explicitly fail a playbook task with a custom error message. It is helpful when you want to stop execution under certain conditions, such as invalid input, missing prerequisites, or failed validations*
```
- name: my first playbook
  hosts: node001:node002
  tasks:
    - name: exit ansible
      fail:
        msg: "EXITED"
```
*tasks after fail module will not be excuted.*

### shell module

*the `shell` module is used to execute shell commands on target systems. Unlike the command module (which runs commands directly without a shell), the shell module processes commands through /bin/sh (or the user's default shell)*
```
- name: my first playbook
  hosts: node001:node002
  tasks:
    - name: get hostname
      shell:
        cmd: hostname
```
*running the above playbook will run hostname command on the target hosts. but it will not return anything to you. if you want to see returns you should add something to playbook like below:*
```
- name: my first playbook
  hosts: node001:node002
  tasks:
    - name: get hostname
      shell:
        cmd: hostname
      register: hn

    - name: show hn output
      debug:
        var: hn
```
*now it will return something like below:*
```
ok: [node001] => {
    "hn": {
        "changed": true,
        "cmd": "hostname",
        "delta": "0:00:00.015062",
        "end": "2025-04-17 10:55:07.631981",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2025-04-17 10:55:07.616919",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "localhost.localdomain",
        "stdout_lines": [
            "localhost.localdomain"
        ]
    }
}
ok: [node002] => {
    "hn": {
        "changed": true,
        "cmd": "hostname",
        "delta": "0:00:00.012748",
        "end": "2025-04-17 10:55:07.850325",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2025-04-17 10:55:07.837577",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "localhost.localdomain",
        "stdout_lines": [
            "localhost.localdomain"
        ]
    }
}
```
*you can change the playbook like below to return only the stdout to you:*
```
- name: my first playbook
  hosts: node001:node002
  tasks:
    - name: get hostname
      shell:
        cmd: hostname
      register: hn

    - name: show hn output
      debug:
        var: hn.stdout
```
*running above playbook will return below:*
```
ok: [node001] => {
    "hn.stdout": "localhost.localdomain"
}
ok: [node002] => {
    "hn.stdout": "localhost.localdomain"
}
```
*you can change the directory of running shell command by adding `chdir` to `args` like below:*
```
- name: my first playbook
  hosts: node001:node002
  tasks:
    - name: get list in home dir
      shell:
        cmd: ls
      register: list
      args:
        chdir: /home

    - name: show list output
      debug:
        var: list.stdout
```
*the above playbook will return the list of files and directories in `/home` path in target hosts*

*`chdir` can be written like below too:*
```
- name: my first playbook
  hosts: node001:node002
  tasks:
    - name: get list in home dir
      shell:
        cmd: ls
        chdir: /home
      register: list

    - name: show list output
      debug:
        var: list.stdout
```

### ansible-doc command
*`ansible-doc <MODULE_NAME>` command will return description about MODULE. like:*\
`ansible-doc shell` *will return description about shell module*

### Variables

*variables can be defined like below in playbook:*
```
---
- name: intro to variables
  hosts: all
  vars:
    favecolor: Blue
  tasks:
    - name: the the variable
      debug:
        var: favecolor
```
*in above playbook the `debug` module will return value of `favecolor` variable which is `Blue`*

*to show value of variable in `msg` not `var` in debug module :*
```
---
- name: intro to variables
  hosts: all
  vars:
    favecolor: Blue
  tasks:
    - name: the the variable
      debug:
        msg: "my favorite color is {{ favecolor }}"
```
*i mean you should use put the variable in `{{  }}` (with 2 spaces) to use it in msg*

*you can define vars in a file and use it in playbook file like below :*\
*for exmple the file below (`first-vars.yml`) (the vars file should be yml) :*
```
---
favecar: Lexus
```
*and the playbook is like below:*
```
- name: intro to variables
  hosts: all
  vars_files:
    - ./first-vars.yml
  tasks:
    - name: the the variable
      debug:
        msg: "my favorite car is {{ favecar }}"
```
### dictionaries in var files

*you can add dictionaries like below in your var file:*
```
---

faveteams:
  Iran: Esteghlal
  England: Chelsea
```
*and use it in playbook like below :*
```
---
- name: intro to variables
  hosts: node001
  vars_files:
    - ./first-vars.yml
  tasks:
    - name: the the variable
      debug:
        msg: "my favorite teams are {{ faveteams }}"
```
*or for just one value in that dictionary :*
```
---
- name: intro to variables
  hosts: node001
  vars_files:
    - ./first-vars.yml
  tasks:
    - name: the the variable
      debug:
        msg: "my favorite teams are {{ faveteams.Iran }}"
```

### lists in var file

*add list to var file like below:*
```
---

servers:
  - node001
  - node002
  - node002
```
*and use it in playbook like below:*
```
---
- name: intro to variables
  hosts: node001
  vars_files:
    - ./first-vars.yml
  tasks:
    - name: the the variable
      debug:
        msg: "my main server is {{ servers[0] }} and i have 2 other servers which are {{ servers[1] }} and {{ servers[2] }}"
```

***everything in `vars_files` can be written in playbook under `vars`***

*to mix (combine) two dictionaries (for example we have two dictionaris named `first_dict` and `second_dict` in vars file):*

```
---
- name: intro to variables
  hosts: node001
  vars_files:
    - ./first-vars.yml
  tasks:
    - name: the the variable
      debug:
        msg: "my best teams are {{ first_dict | combine(second_dict) }}"
```
***be careful*** *if there is a key in first_dict which is repeated in second_dict with another value in the result value of second_dict will be shown*

### Ansible Facts

*`ansibe_facts` is a built-in variable in Ansible that automatically gathers system information (known as "facts") about the target hosts when a playbook runs. like command below which we saw in ***ad hoc commands*** section:*\
`ansible node001 -m gather_facts -i inventory.ini`

*in playbook :*
```
---
- name: ansible facts
  hosts: node001
  tasks:
    - name: show ansible facts
      debug:
        var: ansible_facts.distribution
```
*in last line you can use `var: ansible_facts` which will return a big dictionary of facts about target hosts or use `.` after `ansible_facts` (like `var: ansible_facts.distribution`) to see just a key of that dictionary*

### set_fact module

*In Ansible, the `set_fact` module is used to dynamically create or update variables during playbook execution. Unlike variables defined in `vars:` or `host_vars`, `set_fact` allows you to compute new variables on-the-fly based on conditions, command outputs, or other runtime data.*

* *Creates Host-Specific Variables : Variables are set per host (like host_vars but dynamically).*
* *Persists for the Entire Play : Unlike register (which stores task output), set_fact vars remain available for all subsequent tasks.*
* *Overwrites Existing Variables : If a variable already exists, set_fact will update it.*

*Example : combine 2 dictionaries using `set_fact` and create new dictionary :*
```
---
- name: ansible set_fact
  hosts: node001
  vars:
    dict01:
      name: amir
      family: dehghani
    dict02:
      age: 32
      wy: 14
  tasks:
    - name: create new dict using dict01 and dict02
      set_fact:
        new_dict: "{{dict01 | combine(dict02)}}"

    - name: show newdict
      debug:
        msg: "new dictionary is {{new_dict}}"
```

______________
### just to know :
```
---
- name: show 2 times of num1 and 3 times of num2
  hosts: node001
  vars:
    num1: 32
    num2: 14
  tasks:
    - name: what is 2 times of num1
      set_fact:
        new_var: "{{num1*2}}"

    - name: show result
      debug:
        msg: "{{new_var}} and {{num2*3}}"
```
_____________
### register

*In Ansible, register is a keyword used to capture the output of a task (like a module's execution) and store it in a variable for later use in a playbook. It allows you to:*

* *Save command outputs (e.g., shell, command, or any module).*

* *Inspect results (success/failure, stdout, stderr, rc).*

* *Make decisions (using when conditions).*

* *Reuse data across tasks (e.g., parsing JSON, extracting values).*

***Key Features of register***

* Captures Module Outputs : Stores the return values of a task (e.g., stdout, stderr, rc).

* Works with All Modules : Compatible with command, shell, uri, stat, and even custom modules.

* Structured Data : Output is stored as a Python dictionary (e.g., result.stdout).

* Temporary Scope : The variable exists only for subsequent tasks in the same play.

#### rc :

*In Ansible, when you register the output of a task, the rc (short for "return code") is a key in the resulting variable that indicates the exit status of the executed command or module.*

What Does rc Mean?

* rc stands for Return Code (also called Exit Status in Linux/Unix).

* It is a numeric value (0 for success, non-zero for failure).

* Used to determine if a command/module succeeded or failed.

_____________

### file module

*In Ansible, the `file` module is used to manage files, directories, and symbolic links on target systems. It provides a way to perform common file operations like:*

* ***Create/Delete*** *files or directories.*

* ***Set Permissions*** *(ownership, mode, ACLs).*

* ***Create Symlinks/Hardlinks.***

* ***Modify Attributes*** *(timestamps, SELinux context).*

#### examples:
```
- name: intro to file module
  hosts: node001
  tasks:
    - name: create a new file
      file:
        path: /home/host1/newfile.txt
        mode: '0655'
        state: touch
```
*executing playbook above will create a file named `newfile.txt` in the path `/home/host1/` or if the file exists it will touch it.*

```
- name: intro to file module
  hosts: node001
  tasks:
    - name: create a directory
      file:
        path: /home/host1/new-dir
        mode: '0755'
        owner: host1
        group: host1
        state: directory
```
*executing the playbook above will create a directory named `new-dir` in the path `/home/host1/` and if it exists it will change the owner, group and mode of directory.*

#### become:
*to execute modules or one module in playbook with `sudo` privileges*

```
- name: intro to file module
  hosts: node001
  vars:
    base_dir: /home/host1
  tasks:
    - name: change host1 directory owner and group to root
      file:
        path: "{{base_dir}}"
        owner: root
        group: root
```
*if you run playbook above you will get error below:*\
```
fatal: [node001]: FAILED! => {"changed": false, "gid": 1000, "group": "host1", "mode": "0700", "msg": "chown failed: [Errno 1] Operation not permitted: b'/home/host1'", "owner": "host1", "path": "/home/host1", "secontext": "unconfined_u:object_r:user_home_dir_t:s0", "size": 145, "state": "directory", "uid": 1000}
```
*because you it can't change ownership of files to root without sudo privileges*\
*for that we use `become` like below:*
```
- name: intro to file module
  hosts: node001
  vars:
    base_dir: /home/host1
  tasks:
    - name: change host1 directory owner and group to root
      become: true
      file:
        path: "{{base_dir}}"
        owner: root
        group: root
```
*after executing the playbook above i got another error:*
```
fatal: [node001]: FAILED! => {"msg": "Missing sudo password"}
```
*to fix this error i hade to add line below to `/etc/sudoers` file in target host:*
```
host1 ALL=(ALL) NOPASSWD:ALL
```
*after this change and running the playbook above, owner and group of `/home/host1` directory in target host changed to root*

*if you use `become: true` like below:*
```
- name: intro to file module
  hosts: node001
  become: true
  vars:
    base_dir: /home/host1
  tasks:
    - name: change host1 directory owner and group to root
      file:
        path: "{{base_dir}}"
        owner: root
        group: root
```
*the playbook will do all tasks with sudo privileges but if we use in one task like it was before, only that task will be executed with sudo privileges*

#### delete a file with file module
```
- name: intro to file module
  hosts: node001
  vars:
    base_dir: /home/host1
  tasks:
    - name: delete the file
      file:
        path: "{{base_dir}}/newfile.txt"
        state: absent
```
*using `state: absent` you can delete a file.*
__________________
***TIP:*** *if you want to use a variable in `path:` like above playbook, you should write path in `""` and write the variable in `{{}}` (see last playbook)*
__________________

### ignore_errors

*In Ansible, the ignore_errors directive is used to force a playbook to continue executing even if a task fails. By default, Ansible stops all tasks on a host if any task fails, but ignore_errors: yes overrides this behavior.*

***Key Features*** of `ignore_errors`
|***Behavior***|***Default (Without `ignore_errors`***)	|***With `ignore_errors: yes`***|
|--------------|----------------------------------------|-------------------------------|
|***Task Fails***|Playbook stops (host is marked as failed).|Playbook continues (error is logged but ignored).|
|***Use Case***|Critical tasks (e.g., package installation).|Non-critical tasks (e.g., optional checks).|
|***Error Handling***|Requires manual recovery (e.g., rescue).|Automatically skips failure.|
-----------------------
*for example in playbook below:*
```
- name: intro to file module
  hosts: node001
  vars:
    base_dir: /home/host1
  tasks:
    - name: change host1 directory owner and group to root
      file:
        path: "{{base_dir}}"
        owner: root
        group: root
        state: directory
    - name: create the file
      file:
        path: "{{base_dir}}/newfile.txt"
        state: touch
```
*because we didn't use `become: true` for first task it will be failed and the second task won't be executed too. but we can use `ignore_errors` for first task to let second task be done anyway. like below:*
```
- name: intro to file module
  hosts: node001
  vars:
    base_dir: /home/host1
  tasks:
    - name: change host1 directory owner and group to root
      file:
        path: "{{base_dir}}"
        owner: root
        group: root
        state: directory
      ignore_errors: true
    - name: create the file
      file:
        path: "{{base_dir}}/newfile.txt"
        state: touch
```
*we can debug the first task using `debug` module like below:*
```
- name: intro to file module
  hosts: node001
  vars:
    base_dir: /home/host1
  tasks:
    - name: change host1 directory owner and group to root
      file:
        path: "{{base_dir}}"
        owner: root
        group: root
        state: directory
      ignore_errors: true
      register: output
    - name: create the file
      file:
        path: "{{base_dir}}/newfile.txt"
        state: touch
    - name: debug first task
      debug:
        var: output
```
*in playbook above the debug module will write the output of first task. which will be like below:*
```
ok: [node001] => {
    "output": {
        "changed": false,
        "failed": true,
        "gid": 1000,
        "group": "host1",
        "mode": "0700",
        "msg": "chown failed: [Errno 1] Operation not permitted: b'/home/host1'",
        "owner": "host1",
        "path": "/home/host1",
        "secontext": "unconfined_u:object_r:user_home_dir_t:s0",
        "size": 161,
        "state": "directory",
        "uid": 1000
    }
}
```
*if you look carefully you will see that the `failed` key in output is `true` so you can use `when:` in `debug` module to ask `debug` module to give the output when the first task is failed. like below:*
```
- name: intro to file module
  hosts: node001
  vars:
    base_dir: /home/host1
  tasks:
    - name: change host1 directory owner and group to root
      file:
        path: "{{base_dir}}"
        owner: root
        group: root
        state: directory
      ignore_errors: true
      register: output
    - name: create the file
      file:
        path: "{{base_dir}}/newfile.txt"
        state: touch
    - name: debug first task
      debug:
        var: output
      when: output.failed
```
another example of **when** :
```
- name: too use when in debug for shell module
  hosts: node001
  tasks:
    - name: create a file
      shell:
        cmd: touch /root/new.txt
      ignore_errors: true
      register: result
    - name: show the result
      debug:
        var: result
      when: result.rc != 0
```
*the `debug` module in playbook above will show the result when the `rc` of `shell` module is not 0 ( if it is 0 it means it is not failed )*


### copy module

*In Ansible, the `copy` module is used to transfer files from the local control machine (or the Ansible controller) to remote hosts. It can also copy content directly from variables (inline text) to files on remote systems.*

```
- name: copy playbook directory to node001
  hosts: node001
  vars:
    src_dir: "{{playbook_dir}}"
  tasks:
    - name: copy what i said in name of file
      copy:
        src: "{{src_dir}}"
        dest: /home/host1
        owner: amir
        group: amir
        mode : '0644'
```
*as you see the `copy` module will copy the `src` to `dest` and it can change its owner, group and mode (these are optional)*
__________________
***playbook_dir*** is the directory which this playbook is in it. *this is a `built-in magic` variable of ansible*
__________________
```
- name: copy content to a file in node001
  hosts: node001
  tasks:
    - name: copy what i said in name of file
      copy:
        content: "this content will be copied to the file in dest"
        dest: /home/host1/newfile.txt
```
*running this playbook will copy `content:` to `dest:` in target host*

### sync module

*In Ansible, the `synchronize` module is used to efficiently synchronize files and directories between the control machine (Ansible controller) and remote hosts (or between remote hosts) using `rsync` under the hood. It’s ideal for large-scale file transfers, recursive directory syncing, and minimizing network traffic with delta transfers.*

_______________
**Tip :** *`rsync` is not a built-in module. ***built-in*** modules name start with `ansible.builtin`. for example `ansible.builtin.shell` and if you use the last part (for example `shell`) as its name, ansible will understand it is built-in. but for not built-in modules like sync we should use their compelete name. for `sync` module the compelete name is : ` ansible.posix.synchronize`
_______________
***examples:***
```
- name: Sync local directory to remote
  ansible.posix.synchronize:
    src: ./app_files/  # Local dir (relative to playbook)
    dest: /opt/app/    # Remote dir
    recursive: yes     # Copy subdirectories
```
```
- name: Pull logs from remote to local
  ansible.posix.synchronize:
    src: /var/log/app/  # Remote dir
    dest: ./backups/    # Local dir
    mode: pull          # Reverse direction (remote → local)
```
```
- name: Sync files from web1 to web2
  ansible.posix.synchronize:
    src: /var/www/html/
    dest: /var/www/html/
    delegate_to: web2.example.com  # Sync from web1 → web2
```
```
- name: Sync but exclude junk files
  ansible.posix.synchronize:
    src: ./project/
    dest: /opt/project/
    exclude:
      - "*.tmp"
      - ".git/"
```






