# Demo: Run Ad-hoc commands

## Create an Ansible config file

```bash
ansible-config init –disabled > ansible_full.cfg
```

## Check smaller Ansible config file

Open the file ansible.cfg in the current directory

## Run ping locally

```bash
ansible localhost -m ping
```

## Run cp file locally

```bash
ansible localhost -m copy -a "src=/Users/tiago.bernardo/ssh-keys/tiberna-key.pem dest=/Users/tiago.bernardo/.ssh/id_rsa"
```

## Change mode of file locally

```bash
ansible localhost -m file -a "path=/Users/tiago.bernardo/.ssh/id_rsa mode=0600"
```

## Run ping on all remote host

```bash
ansible all -m ping
```

## Run copy file on nodes pattern

```bash
ansible nodes -m copy -a "src=/etc/hosts dest=/tmp/hosts"
```

## Install nginx-light

```bash
ansible webserver -m apt -a "name=nginx-light state=present" --become
```

## Check if nginx service is running

```bash
ansible webserver -m command -a "systemctl status nginx"
```

## Stop nginx service with error

```bash
ansible webserver -m service -a "name=nginx state=stopped"
```

## Stop nginx service

```bash
ansible webserver -m service -a "name=nginx state=stopped" --become
```

## Check if nginx service is running after stop

```bash
ansible webserver -m command -a "systemctl status nginx"
```

## Uninstall nginx-light

```bash
ansible webserver -m apt -a "name=nginx-light state=absent" --become
```

## Delete id_rsa file on localhost

```bash
ansible localhost -m file -a "path=/Users/tiago.bernardo/.ssh/id_rsa state=absent"
```

## Check file is deleted

```bash
ansible localhost -m command -a "ls /Users/tiago.bernardo/.ssh"
```

## Use of all inventory default group

```bash
ansible all -m command -a "ls /tmp"
```
