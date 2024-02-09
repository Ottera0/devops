# devops TP3

## Question 3-1: Document your inventory and base commands: 

```yaml
all:
 vars:
   ansible_user: centos
   ansible_ssh_private_key_file: ~/.ssh/id_rsa
 children:
   prod:
     hosts: centos@ewan.cereza.takima.cloud
```

all: This is a special group that includes all the hosts in the inventory file.

`vars`: Here, you define variables that are applicable to all hosts in the inventory.
`ansible_user`: The default SSH user Ansible will use for connecting to the managed nodes. In this case, it's set to centos.
`ansible_ssh_private_key_file`: The path to the private key file that Ansible will use for SSH connections. It's set to `~/.ssh/id_rsa`, which is a common default SSH key location on a Unix/Linux system.
`children`: This is used to define groupings of hosts within the inventory. These can be seen as subgroups under the main group.

`prod`: Represents a group for production servers.
`hosts`: Under this, individual hosts or servers are specified.
`centos@ewan.cereza.takima.cloud`: A specific host in the prod group. The format user@hostname indicates that for this host, connections should be made using the centos user. This overrides the ansible_user for this specific host if the Ansible playbook or command targets it directly.

With this inventory, we can run the following command to ping the server : 
`ansible all -i devops/ansible/inventories/setup.yml -m ping`

## Question 3-2/3-3 : Document your playbook and docker_container configurations:

in the initial playbook, we have the roles : 
```yaml
- hosts: all
  gather_facts: false
  become: true
  vars_files:
    - vars/secret.yml

# Install Docker
  roles:
    - docker
    - network
    - database
    - app
    - proxy


```

This playbook will run the roles in the order they are listed.

In the `docker` role, we run these tasks:
```yaml
---
# tasks file for devops/ansible/roles/docker
  - name: Install device-mapper-persistent-data
    yum:
      name: device-mapper-persistent-data
      state: latest

  - name: Install lvm2
    yum:
      name: lvm2
      state: latest

  - name: add repo docker
    command:
      cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

  - name: Install Docker
    yum:
      name: docker-ce
      state: present

  - name: Install python3
    yum:
      name: python3
      state: present

  - name: Install docker with Python 3
    pip:
      name: docker
      executable: pip3
    vars:
      ansible_python_interpreter: /usr/bin/python3

  - name: Make sure Docker is running
    service: name=docker state=started
    tags: docker
```

This role installs Docker and its dependencies, and ensures that the Docker service is running.

In the `network` role, we run these tasks:
```yaml
---
# tasks file for devops/ansible/roles/network
- name: Create network
  vars:
    ansible_python_interpreter: /usr/bin/python3
  community.docker.docker_network:
    name: app-network
```

This role creates a Docker network named app-network.

In the `database` role, we run these tasks:
```yaml
# tasks file for devops/ansible/roles/database
- name: Create database
  vars:
    ansible_python_interpreter: /usr/bin/python3
  community.docker.docker_container:
    name: database
    image: ottera/devops-database
    networks:
      - name: app-network
```

This role creates a Docker container named database, using the `ottera/devops-database` image from dockerhub, and connects it to the app-network.

In the `app` role, we run these tasks:
```yaml
# tasks file for devops/ansible/roles/database
- name: Create backend container
  vars:
    ansible_python_interpreter: /usr/bin/python3
  community.docker.docker_container:
    name: backend
    image: ottera/devops-backend 
    networks:
      - name: app-network
    env:
      SPRING_DATASOURCE_URL: jdbc:postgresql://database/db
      SPRING_DATASOURCE_USER: usr
      SPRING_DATASOURCE_PASSWORD: pwd
```

This role creates a Docker container named backend, using the `ottera/devops-backend` image from dockerhub, and connects it to the app-network. It also sets environment variables for the Spring Boot application to connect to the database.

In the `proxy` role, we run these tasks:
```yaml
# tasks file for devops/ansible/roles/database
- name: Create httpd container
  vars:
    ansible_python_interpreter: /usr/bin/python3
  community.docker.docker_container:
    name: devops-httpd
    image: ottera/devops-httpd
    networks:
      - name: app-network
    ports:
    - 80:80
```

This role creates a Docker container named devops-httpd, using the `ottera/devops-httpd` image from dockerhub, and connects it to the app-network. It also exposes port 80 to the host.

### Ansible Vault

to setup the vault we run this command : 
`ansible-vault create ansible/vault/secret.yml`

we can then edit it to enter our variables : 
`ansible-vault edit ansible/vault/secret.yml`

Then we can use these variables in our file, for exemple the `setup.yml` : 

```yml
all:
 vars:
   ansible_user: "{{ ansible_user }}"
   ansible_ssh_private_key_file: "{{ ansible_ssh_private_key_file }}"
 children:
   prod:
     hosts: "{{ ansible_hosts }}"
```



Finally, to run the playbook, we use the following command:
`ansible-playbook -i inventories/setup.yml playbook.yml --ask-vault-pass`
