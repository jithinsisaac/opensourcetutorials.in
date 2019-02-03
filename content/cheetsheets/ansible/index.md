---
title: "Ansible"
date: "2018-12-31"
description: "Ansible "
disable_comments: false # Optional, disable Disqus comments if true
authorbox: true # Optional, enable authorbox for specific post
toc: false # Optional, enable Table of Contents for specific post
mathjax: true # Optional, enable MathJax for specific post
categories:
  - "Cheetsheets"
tags:
  - "Cheetsheets"
thumbnail: "img/thumbs/ansible.png"
---
Ansible is powerful open source IT automation tool. It allows you to manage IT infrasture and apps deployment easily.

<!--more-->

## Step 1 - Installing Ansible

To configure the PPA on your machine and install ansible run these commands:

       $ sudo apt-get update
       $ sudo apt-get install software-properties-common
       $ sudo apt-add-repository ppa:ansible/ansible
       $ sudo apt-get update
       $ sudo apt-get install ansible

Check ansible install version

   $ ansible --version

## Step 2 - Configure Hosts

All host information is kept in the hosts files as follows

   $ sudo gedit /etc/ansible/hosts

   $ sudo gedit /etc/ansible/ansible.cfg

   Remove comment of inventory and the sudo_user
   Its default but better to change and make it apply


   $ cat /etc/ansible/hosts

   $ sudo mv hosts hosts.original

   $ sudo gedit hosts

Create entries of the hosts which you are going to manage

         [local]
         localhost

         [itlab6]
         10.0.1.198
         10.0.1.197

Note : Better to use Vi than gedit


Ansible Default configuration
       $ cat /etc/ansible/ansible.cfg

       Keep as it is or you can uncomment following and save
       #inventory      = /etc/ansible/hosts
       #sudo_user      = root

-------------------------------------

## Step 3 - Servers Setup

 ### Install ssh on the server and your host also

   $ sudo apt install ssh

   Check from your computer to server you can connect using the ssh

   $ ssh dbit@10.0.1.198

   This will ask username and password



### This is not necessary to just for additional information
### If you want to enable root login on over ssh then follow following commands

   # gedit /etc/ssh/sshd_config

   Change the line "PermitRootLogin" to "yes"

   # service sshd restart

   $ ssh root@192.168.1.101

   You will be able to connect to the server using root username

### Create ansible user with sudo permissions

On the server where you will be configuring you need to have a user with sudo permissions and authentication method must not need password. i.e use ssh keys

   # sudo adduser ansible
   # sudo usermod -aG sudo ansible

### Setup ssh keys

     On the OUR computer AND WITH ANSIBLE USER ( Create if not not available )
     from where we will be managing ansible

Swith to the user using following command

     $ su ansible

Then start commands

     $ ssh-keygen
       accept all default options

     $ cd /home/ansible/.ssh/
      you can see all keys here at this place

      Copy the Public Key on server where we created ansible user

      $ ssh-copy-id ansible@192.168.1.101
      Input password

     $ ssh-copy-id tayyabali@localhost
     $ ssh-copy-id ansible@localhost
     $ ssh-copy-id ansible@10.0.1.198
     $ ssh-copy-id ansible@10.0.1.197

     Now try

     $ ssh ansible@192.168.1.198
     $ ssh localhost

     You will be logged in without password

### Disable Password Authentication (Recommended) not necessary now
     On Server (Where we have ansible user):
     Now oce you have configured the key based login, now you can disable the password based auth

     $ sudo vi /etc/ssh/sshd_config

     Change follwoing parameter to no as shown below

         PasswordAuthentication no

     $ sudo systemctl reload sshd

### Set Up a Basic Firewall

   $ sudo ufw app list
   $ sudo ufw allow OpenSSH
   $ sudo ufw enable
   $ sudo ufw status


Now lets configure ansible user to run commands as a root on the remote machine

   $ sudo visudo

   add following below the User privilege section, just below the root
   ansible  ALL=(ALL) NOPASSWD: ALL

   CTRL + O  ----> Enter to Save  and CTRL + X


   and then check if its ok
   $ sudo visudo -c

   Check if you can be root without the password

   $ su ansible -

-------------------------------------

`Explore the documentation of the ansible.com`

## Running Ansible Commands

Make sure you are anisible user Also you have configured ansible user on remote hosts with ssh keys login and sudo commands permissions


Check nodes present in the inventory by using folowing command

    $ cat /etc/ansible/hosts


## To ping all hosts

    $ ansible all -m ping

    -m Use following module
    ping is a module

## To list all files on the nodes

    $ ansible all -a "ls -al /home/ansible"

## To open syslog file
    $ ansible all -a "cat /var/log/syslog"

## Copy files to remote machines
On My computer

touch test.txt
Enter some text

$ ansible all -m copy -a "src=test.txt dest=/tmp/test.txt"

## Install packages

$ ansible ubuntu -m apt -a "name=elinks state=latest"

Whats the difference

$ ansible ubuntu -m apt -a "name=elinks state=present"

Will it run ?

    $ ansible ubuntu -s -m apt -a "name=elinks state=present"

-s is become super user and then install


    $ ansible ubuntu -s -m apt -a "name=elinks state=absent"

Above command will remove the package


Lets create a user with the user module

    $ ansible centos -s -m user -a "name=test"

Check test user on the server , see its home directory

    $ ansible centos -s -m user -a "name=test state=abset"

Check User has gone

    $ cat /etc/passwd | grep test

    Assignment install, upgrade ubunutu
    install vlc, install kazam


## Playbook Structure

In the ansible user
      $ mkdir ansible

Use following command to Check the playbook syntex

    $ ansible-playbook name.yaml

Execute the following Playbook

From the URL  :

        https://github.com/ansible/ansible-examples

Execute and Explain the Structure of the playbook


## Gathering the facts



#### Observe the output

        $ ansible centos -m setup

#### You can do grep also

          $ ansible centos -m setup | grep ipv4

          $ ansible localhost -m setup -a 'filter=*ipv4*'


#### To get the facts in to the directory and in separate files

          $ ansible all -m setup -m --tree facts
          $ cd facts
          $ ls

#### Install httpd on ubuntu
          --- # This is a structure example to install httpd on ubuntu

          - hosts: ubuntu
            remote_user: ansible
            become: yes
            become_method: sudo
            connection: ssh
            gather_factcs: yes
            vars:
              userName:myuser
            tasks:
            - name: Install HTTPD server on ubunutu server
              apt: name:apache2 state:latest
              notify:
              - startservice
            handlers:
             - name : startservice
               service: name:apache2 state:restarted


               ---

               - hosts: ubuntu # hosts to be configured

               sudo: yes # sudo permissions

               tasks:

               - name: update

               apt: update_cache=yes

               - name: install apache2

               apt: name=apache2 state=present

               - name: restart apache2

               service: name=apache2 state=restarted

               - debug: msg="apache has been installed"

               ...
