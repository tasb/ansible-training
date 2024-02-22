# Lab 05 - Author your first Ansible Role

## Objectives

- Understand the role structure
- Create a role
- Use the role in a playbook
- Understand the role variables
- [Optional] Make the role available on a Git repository and use it in a playbook

## Prerequisites

- [ ] Create a folder named `lab05` inside `ansible-labs`
- [ ] Navigate to `lab05` folder
- [ ] Copy the `inventory` folder from `lab04` to `lab03`
- Command: `cp -r ../lab04/inventory ./inventory`

## Guide

### Step 01: Understand the role structure

A role is a way of organizing tasks, handlers, variables, and other files in a structured way.

When you create a role, you create a directory structure that contains the role's tasks, handlers, and other files.

Let's create a role named `lab.usersandgroups`:

```bash
ansible-galaxy role init --init-path=./roles/ lab.usersandgroups
```

This command will create a directory structure for the role in the `roles` folder.

Navigate to the `roles/lab.usersandgroups` folder and check the files created.

You must see the following files and directories:

```bash
.
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```

## Step 02: Update documentation and meta files

Update the `README.md` file with the role description and usage with the content available on this [README.md](lab05/README.md) file.

Update the `meta/main.yml` file with the role metadata with following content:

```yaml
galaxy_info:
  role_name: usersandgroups
  author: Your Name
  description: Manage users and groups on Linux/Unix systems
  company: Your Company or Organization

  # List of platforms you support and the versions, e.g., EL (Enterprise Linux) 7, 8, Ubuntu 18.04 (Bionic)
  platforms:
    - name: Rocky
      versions:
        - 8.0
        - 9.0
    - name: Ubuntu
      versions:
        - bionic
        - focal
    - name: Debian
      versions:
        - stretch
        - buster
        - bullseye

  # List of categories this role falls into, from Galaxy's predefined list
  galaxy_tags:
    - system
    - users
    - groups
    - management

  # Dependencies are other roles that this role depends upon to function
  dependencies: []

# Specify minimum Ansible version
min_ansible_version: 2.9

# License as applicable to your role
license:
  - MIT

# Optional: If your role has a README.md file, you can specify its name here. The default is README.md.
readme: README.md
```

## Step 03: Create variables file

Edit the `vars/main.yml` file with the following content:

```yaml
---
# Default list of users to manage
users_list:
  - name: exampleuser
    state: present
    password: "{{ 'examplepassword' | password_hash('sha512') }}"
    groups: ["examplegroup"]
    shell: "/bin/bash"
    createhome: yes
    remove: no
    ssh_key: ""

# Default list of groups to manage
groups_list:
  - name: examplegroup
    state: present

# Default values for user management
default_shell: "/bin/bash"
create_home: yes
remove_home: no
default_folders:
  - "Documents"
  - "Downloads"
  - "Music"
  - "Pictures"
  - "Videos"


# SSH key options
manage_ssh_keys: false
ssh_key_dir: ".ssh"
authorized_keys_file: "authorized_keys"
```

Please go through the file and understand the variables defined.

These list are the parameters that you can use to manage users and groups.

Check the password hash generation using the `password_hash` filter: `{{ 'examplepassword' | password_hash('sha512') }}`.

This filter will generate the password hash for the user `exampleuser` using the `sha512` algorithm.

For now we're setting this sensitive information in plain text, but in a real-world scenario you should use Ansible Vault to encrypt this information.

We'll go through Ansible Vault in the next lab.

## Step 04: Create tasks file

Now, let's edit the tasks file with the following content:

```yaml
---
# Create groups
- name: Ensure groups are present or absent
  ansible.builtin.group:
    name: "{{ item.name }}"
    state: "{{ item.state }}"
  loop: "{{ groups_list }}"

# Create users
- name: Ensure users are present or absent
  ansible.builtin.user:
    name: "{{ item.name }}"
    state: "{{ item.state }}"
    password: "{{ item.password }}"
    groups: "{{ item.groups }}"
    shell: "{{ item.shell }}"
    createhome: "{{ item.createhome }}"
    remove: "{{ item.remove }}"
    home: "{{ item.home }}"
  loop: "{{ users_list }}"

# Manage SSH keys for users
- name: Ensure SSH directories exist
  ansible.builtin.file:
    path: "/home/{{ item.name }}/{{ ssh_key_dir }}"
    state: directory
    owner: "{{ item.name }}"
    group: "{{ item.name }}"
    mode: '0700'
  loop: "{{ users_list }}"
  when: manage_ssh_keys

- name: Add authorized keys
  ansible.builtin.authorized_key:
    user: "{{ item.name }}"
    state: present
    key: "{{ item.ssh_key }}"
  loop: "{{ users_list }}"
  when: item.ssh_key | length > 0
  notify: reload sshd
```

Now let's understand the tasks:

- The first task will create the groups defined in the `groups_list` variable.
- The second task will create the users defined in the `users_list` variable.
- The third task will ensure the SSH directories exist for the users.
- The fourth task will add the authorized keys for the users.
- The `notify` keyword will trigger the `reload sshd` handler when the user is created or modified.
- This handler will restart the SSH service to apply the changes only when the task returns a changed status, meaning we only restart the service when we make changes to authorized keys.

## Step 05: Create handlers file

Now, let's edit the `handlers/main.yml` file with the following content:

```yaml
---
# Optional: Reload SSH daemon if necessary
- name: Reload SSH daemon
  ansible.builtin.service:
    name: sshd
    state: reloaded
  when: manage_ssh_keys
  listen: "reload sshd"
```

## Step 06: Use the role in a playbook

Create a playbook named `playbook.yml` and create a first play to generate SSH keys for the users.

```yaml
---
- name: Generate SSH keys
  hosts: localhost
  become: yes
  tasks:
    - name: Generate SSH keys
      ansible.builtin.openssh_keypair:
        path: "./new_key"
        size: 2048
        state: present
```

Now, let's create a second play to use the role `lab.usersandgroups` to manage the users and groups.

```yaml
---
- name: Manage users and groups
  hosts: all
  become: yes
  roles:
    - role: lab.usersandgroups
      when: ansible_os_family == "RedHat"
  vars:
    users_list:
      - name: adminuser
        state: present
        password: "{{ 'password123' | password_hash('sha512') }}"
        groups: ["wheel"]
        ssh_key: "{{ lookup('file', './new_key') }}"
      - name: devuser
        state: present
        password: "{{ 'password123' | password_hash('sha512') }}"
        groups: ["wheel", "developers"]
        ssh_key: "{{ lookup('file', './new_key') }}"
        default_folders:
          - "Documents"
          - "Downloads"
          - "Work"
      - name: testeruser
        state: present
        password: "{{ 'password123' | password_hash('sha512') }}"
        groups: ["testers"]
        ssh_key: "{{ lookup('file', './new_key') }}"
        default_folders:
          - "Documents"
          - "Downloads"
          - "Tests"
    groups_list:
      - name: developers
        state: present
      - name: testers
        state: present
```

