Role Name: Ansible NFS Server
=========

The role deploys an NFS server to host.

Requirements
------------

Ubuntu 22

Role Variables
--------------

Defaults/main.yml contains all variable you can set. you can set these variable to your own in your inventory or vars folder.

Example Playbook
----------------

    - hosts: all
      become: yes
      roles:
        - ansible-nfs-server

