# Lab 07 - Using Ansible Vault and create a complex scenario

## Objectives

- Create Ansible Roles from existing playbook
- Use Ansible Vault to encrypt sensitive data
- Use Templates with Jinja2 instead of regex replace

## Prerequisites

- [ ] Create a folder named `lab07` inside `ansible-labs`
- [ ] Navigate to `lab07` folder
- [ ] Copy all content from `lab04` fodler to `lab07` folder
  - Command: `cp -r ../lab04/* .`

## Guide

In this lab, you will use the playbook created on `lab04`and create roles for each part of the playbook.
You will also use Ansible Vault to encrypt sensitive data and use Jinja2 templates to update `redis.conf` file.

### Step 01: Create a role for each part of the playbook

On the playbook created on `lab04`, you have one play for Redis, another for PostgreSQL and another for Apache.

Look into this playbook and create a role for each part of it.

Then update the playbook to use the roles you've created.

Your playbook should look like this:

```yaml
---
- name: Install Redis, PostgreSQL and Apache
  hosts: all
  become: yes
  roles:
    - lab.redis
    - lab.postgresql
    - lab.apache
```

### Step 02: Use template for `redis.conf`

On the `lab.redis` role, you should have a task with the following content:

```yaml
- name: Configure Redis
  ansible.builtin.replace:
    path: /etc/redis/redis.conf
    regexp: '^# requirepass foobared'
    replace: 'requirepass {{ redis_password }}'
    backup: yes
  notify:
    - Restart Redis
```

This is a great opportunity to use a template instead of regex replace.

Create a template file named `redis.conf.j2` inside `templates` folder with the content of this file: [redis.conf]


### Step 03: Create a template file

Create a file named `index.html.j2` inside `templates` with the following content:

```html
<!DOCTYPE html>

<html>
  <head>
    <title>Welcome to {{ ansible_hostname }}</title>
  </head>
  <body>
    <h1>Welcome to {{ ansible_hostname | upper }}</h1>
    <p>This is a sample page created using Jinja2 template.</p>
    <p>Version: {{ version }}</p>
    <p>Date: {{ ansible_date_time.date }}</p>
  </body>
</html>
```

Take a look at the variables used inside the template file. You can use any variable available in Ansible to create a dynamic template.

On this example, you are using the following variables:

- `ansible_hostname | upper`: The hostname of the machine in uppercase
- `version`: The version of `nginx` to be installed
- `ansible_date_time.date`: The current date

### Step 04: Update `tasks/main.yml`

Update the `tasks/main.yml` file removing all content and to include the following content:

```yaml
---
- name: Install nginx
  package:
    name: nginx={{ version }}
    state: present

- name: Start nginx
  service:
    name: nginx
    state: started
    enabled: yes

- name: Create index.html
  template:
    src: "{{ template_file }}"
    dest: /usr/share/nginx/html/index.html
  notify: Restart nginx
```

### Step 05: Update `handlers/main.yml`

Update the `handlers/main.yml` file removing all content and to include the following content:

```yaml
---
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

### Step 06: Create a playbook

Create a playbook named `nginx.yml` with the following content:

```yaml
---
- name: Install nginx
  hosts: all
  become: yes
  roles:
    - lab.nginx
```

### Step 07: Run the playbook

Run the playbook using the following command:

```bash
ansible-playbook -i inventory nginx.yml
```

### Step 08: Access the server

Open a web browser and access the server using the following URL:

- [http://servidor-0](http://servidor-0)
- [http://servidor-1](http://servidor-1)

You should see the `index.html` file created using the Jinja2 template.

For each server, the hostname and the date should be different.

## Conclusion

You've learned how to use Jinja2 templates with Ansible. You've created a role that uses a template file to create a dynamic `index.html` file for `nginx` server.
