# Lab 03 - Author and run your first playbook

## Contents

- [Objective](#objective)
- [Prerequisites](#prerequisites)
- [Guide](#guide)
  - [Step 1: Create the Playbook](#step-1-create-the-playbook)
  - [Step 2: Run the Playbook](#step-2-run-the-playbook)
  - [Step 3: Test the Web Server](#step-3-test-the-web-server)
  - [Step 4: Update the Playbook](#step-4-update-the-playbook)
  - [Step 5: Change homepage](#step-5-change-homepage)
  - [Step 6: Add a smoke test](#step-6-add-a-smoke-test)
- [Conclusion](#conclusion)

## Objective

Create an Ansible playbook that installs and configures a web server on a managed node.

## Prerequisites

- [ ] Create a folder named `lab03` inside `ansible-labs`
- [ ] Navigate to `lab03` folder
- [ ] Copy the `inventory` folder from `lab02` to `lab03`

## Guide

### Step 1: Create the Playbook

Create a file named `webserver.yml` inside `lab03` folder.

Add the following content to the file:

```yaml
---
- name: Install and configure web server
  hosts: webserver
  become: true
  tasks:
    - name: Install Apache
      ansible.builtin.yum:
        name: httpd
        state: latest
    - name: Start Apache
      ansible.builtin.service:
        name: httpd
        state: started
```

On this playbook, we are:

- Installing Apache web server
- Starting Apache web server
- Setting the `become` option to `true` to run the tasks as root
- Setting the `hosts` option to `webserver` to run the tasks on the hosts in the `webserver` group

### Step 2: Run the Playbook

Run the playbook using the `ansible-playbook` command:

```bash
ansible-playbook -i inventory/inventory.yml webserver.yml
```

This will run the playbook on the hosts in the `webserver` group.

You should see output similar to the following:

```bash
PLAY [Install and configure web server] ****************************************************

TASK [Gathering Facts] *********************************************************************
ok: [servidor-0]

TASK [Install Apache] **********************************************************************
changed: [servidor-0]

TASK [Start Apache] ************************************************************************
changed: [servidor-0]

PLAY RECAP *********************************************************************************
servidor-0          : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Step 3: Test the Web Server

Run the following command to test the web server:

```bash
curl http://servidor-0
```

You should see output similar to the following:

```bash
curl: (7) Failed to connect to servidor-0 port 80 after 5 ms: Couldn't connect to server
```

This is expected because we haven't opened port 80 on the firewall.

Let's use Ansible to open the port on the firewall.

### Step 4: Update the Playbook

Update the `webserver.yml` file to add a new task to open port 80 on the firewall.

Add the following content to the file after the `Start Apache` task:

```yaml
    - name: Open port 80 on firewall
      ansible.builtin.firewalld:
        port: 80/tcp
        permanent: true
        state: enabled
    - name: Reload firewalld
      ansible.builtin.service:
        name: firewalld
        state: reloaded
```

Pay attention to the indentation. The tasks should be at the same level as the `Start Apache` task.

Run the playbook again:

```bash
ansible-playbook -i inventory/inventory.yml webserver.yml
```

You should see output similar to the following:

```bash
PLAY [Install and configure web server] ****************************************************

TASK [Gathering Facts] *********************************************************************
ok: [servidor-0]

TASK [Install Apache] **********************************************************************
ok: [servidor-0]

TASK [Start Apache] ************************************************************************
ok: [servidor-0]

TASK [Open port 80 on firewall] ************************************************************
changed: [servidor-0]

TASK [Reload firewalld] **********************************************************************
changed: [servidor-0]

PLAY RECAP *********************************************************************************
servidor-0          : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Check that in the 2 tasks that we ran before, you get a `ok` status instead of `changed`. This is because the tasks were already run before and there was no need to run them again.

Now let's test the web server again:

```bash
curl http://servidor-0
```

On the output, you should see the HTML code of the default Apache page.

You can also open the URL on your browser to see the page.

### Step 5: Change homepage

First, let's create a new file named `index.html` inside `lab03` folder.

Add the following content to the file:

```html
<html>
  <head>
    <title>Ansible Lab</title>
  </head>
  <body>
    <h1>Ansible Lab</h1>
    <p>This is a test page</p>
  </body>
</html>
```

Now, let's update the playbook to copy the file to the web server.

Update the `webserver.yml` file to add a new task to copy the file to the web server.

Add the following content to the file after the `Reload firewalld` task:

```yaml
    - name: Copy index.html
      ansible.builtin.copy:
        src: index.html
        dest: /var/www/html/index.html
```

Pay attention to the indentation. The task should be at the same level as the `Reload firewalld` task.

Run the playbook again:

```bash
ansible-playbook -i inventory/inventory.yml webserver.yml
```

You should see output similar to the following:

```bash
PLAY [Install and configure web server] **************************************

TASK [Gathering Facts] *******************************************************
ok: [servidor-0]

TASK [Install Apache] ********************************************************
ok: [servidor-0]

TASK [Start Apache] **********************************************************
ok: [servidor-0]

TASK [Open port 80 on firewall] **********************************************
ok: [servidor-0]

TASK [Reload firewalld] ********************************************************
changed: [servidor-0]

TASK [Copy index.html] *******************************************************
changed: [servidor-0]

PLAY RECAP ************************************************************************************************
servidor-0          : ok=6    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Check that task `Reload firewalld` continues to be `changed` because we are reloading every time the firewall service.

Now navigate to the URL <http://servidor-0> and you should see the new page.

### Step 6: Add a smoke test

Update the `webserver.yml` file to add a new task to run a smoke test.

Add the following content to the `webserver.yml` file after the `Copy index.html` task:

```yaml
    - name: Run smoke test
      ansible.builtin.uri:
        url: http://servidor-0
        return_content: yes
      register: result
    - name: Debug smoke test
      ansible.builtin.debug:
        msg: "{{ result.content }}"
```

Let's run the playbook again:

```bash
ansible-playbook -i inventory/inventory.yml webserver.yml
```

## Conclusion

In this lab, we created our first playbook to install and configure a web server. You also learned how to run a playbook and how to update it to add new tasks.