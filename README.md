# Docker Image Build and Push to DockerHub using Ansible

Playbook to insatll Docker and build a python application image from a dockerfile and then it pushed to your  Docker Hub account.

## Ansible Modules used

- yum
- pip
- service
- git
- docker_image
- docker_login


## How to Use

```
git clone https://github.com/BetcyBabu/ansible-build-push-dockerfile.git
cd ansible-build-push-dockerfile
ansible-playbook dockerimage-build.yml
```


## Behind the code

```
---
- name: "Creating Docker Image"
  hosts: localhost
  become: true
  vars:
    git_url: "https://github.com/BetcyBabu/Dockerizing-a-Python-web-app.git"
    docker_username: betcybabu                     # Docker Hub username
    docker_password: "*******"                     # Docker Hub password
    image_name: betcybabu/flaskapp
  tasks:
    - name: "Installing Docker and the required softwares"
      yum:
        name:
          - docker
          - git
          - python2-pip
        state: present
    - name: "Starting Docker service"
      service:
        name: docker
        state: restarted
        enabled: true
    - name: "Installing Docker client for Python"
      pip:
        name: docker-py
        state: present
    - name: "Cloning git repository"
      git:
        repo: "{{ git_url}}"
        dest: /var/flaskapp
      register: git_status
    - name: "Login into Image repository"
      when: git_status.changed == true
      docker_login:
        username: "{{ docker_username }}"
        password: "{{docker_password}}"
    - name: "Build Docker image from Dockerfile"
      when: git_status.changed == true
      docker_image:
        build:
          path: /var/flaskapp/
        name: "{{image_name}}"
        source: build
        push: yes
        tag : "{{ item }}"
      with_items:
        - latest
        - "{{git_status.after}}"
```

## Image created from playbook

```
[root@ip-172-31-10-158 ~]# docker image ls
REPOSITORY           TAG                                        IMAGE ID       CREATED         SIZE
betcybabu/flaskapp   734e3125c32627edb65ebf92e7b1a5d65bd66d36   c385f84f9849   3 minutes ago   62.7MB
betcybabu/flaskapp   latest                                     c385f84f9849   3 minutes ago   62.7MB
alpine               3.8                                        c8bccc0af957   23 months ago   4.41MB
[root@ip-172-31-10-158 ~]#

````

## Playbook Output

![image](https://user-images.githubusercontent.com/23291976/148966085-72ec1045-2aa3-4812-8453-8e740c3ec092.png)

## From Dockerhub

![image](https://user-images.githubusercontent.com/23291976/148966178-98715bd5-424d-4129-8d53-14deaa354f53.png)

