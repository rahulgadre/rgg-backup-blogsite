---
title: Docker-Ansible-AWS
date: 2019-01-30 4:28:47 +07:00
# modified: 2020-07-11 16:49:47 +07:00
tags: [aws]
description: Deploying Docker containers using Ansible on AWS
---

As I have been regularly using Docker 🐳, Ansible, and AWS, I wanted to use all 3 technologies together. Multiple Docker containers can be deployed using Docker compose. However, the Docker Ansible modules also let you deploy multiple containers using a playbook. I wanted to try it so I deployed Docker containers using Ansible on AWS. The example written below contains 2 Docker containers (drupal & postgres) deployed on an EC2 container which has Ansible running on it.  

#### Ansible Playbook:

``` yml
---

-  name: Using Ansible to deploy containers
   hosts: localhost
   gather_facts: no

   tasks:

   - name: Deploying Drupal container
     docker_container:
       name: drupal
       image: drupal
       ports:
         - "80:80"
       volumes:
         - drupal-modules:/var/www/html/modules
         - drupal-profiles:/var/www/html/profiles
         - drupal-themes:/var/www/html/themes
         - drupal-sites:/var/www/html/sites

   - name: Deploying postgres container
     docker_container:
       name: database
       image: postgres
       ports:
         - "5432:5432"
       env:
         POSTGRES_PASSWORD: secret
```

Just like it would with Docker compose, the drupal site was up and running but this time using Docker containers deployed using Ansible.

![](/assets/img/drupal.JPG)


Issues faced:

- As I didn't have to expose the Postgres port (5432) in Docker compose, I wasn't sure about exposing this port in the playbook. Therefore, I had to expose this port while deploying the postgres container. 

- Since I had done this using Docker for Windows and Docker for Mac, I never had to open any port as there was no firewall. However, I needed to open the postgres port in the AWS security group as the host used in this scenario was in AWS.   

- While setting up the Drupal site, it showed an option to type in the IP/name of the host hosting the postgres database. localhost was set by default. While using Docker compose, the postgres service name was used. However, the public IP address of the EC2 instance hosting the Docker containers needed to be used in this scenario. 
