host_vars
=========

1. Theory: What is host_vars?

* host_vars is used to define variables for a specific inventory host. It is useful when servers in the same group need slightly different configurations.

* Imagine you have two EC2 instances in your development group:

      Server	    Application port
      
      server01	    8081
      server02	    8082

* Both servers belong to dev, so they share common settings. But each server needs a different port.
  
      - group_vars/dev.yml stores settings shared by all development servers.
      - host_vars/server01.yml stores settings specific to server01.
      - host_vars/server02.yml stores settings specific to server02.

* Ansible can load these variables automatically when the files are in the expected location relative to your inventory or playbook.

Practical task: Create host_vars:

* Assume your Ansible project directory looks like this:

      ansible-lab/
      ├── inventory
      ├── deploy.yml
      ├── group_vars/
      │   └── dev.yml
      └── host_vars/
          ├── server01.yml
          └── server02.yml

Step 1 — Check your inventory

Your inventory file:

    [dev]
    server01
    server02

Step 2 — Keep shared variables in group_vars/dev.yml

    app_name: Empower
    app_version: "FR6"
    environment: dev
    port: 8080

Step 3 — Create host_vars/server01.yml

    port: 8081
    server_role: primary

Step 4 — Create host_vars/server02.yml

    port: 8082
    server_role: secondary

Notice that we define port in both places. This is intentional: the host-specific value takes precedence over the group-level value for that host.

3. Test with a playbook

Create deploy.yml:


    ---
    - name: Display host-specific application settings
      hosts: dev
    
      tasks:
        - name: Display application configuration
          ansible.builtin.debug:
            msg: >-
              Host={{ inventory_hostname }}, 
              App={{ app_name }},
              Version={{ app_version }},
              Environment={{ environment }},
              Port={{ port }},
              Role={{ server_role }}

* inventory_hostname is a built-in Ansible variable that contains the name of the host as it appears in your inventory file.
* You don't need to define it yourself. Ansible provides it automatically.
* {{ inventory_hostname }} — Jinja2 syntax that inserts the current inventory host's name
* When Ansible runs the task on server01, it substitutes the variable with server01. When it runs on server02, it substitutes it with server02.
    
Run:

    ansible-playbook -i inventory deploy.yml

Expected results:

    - server01 should use port 8081 and role primary.
    - server02 should use port 8082 and role secondary.
    - Both should use application Empower, version FR6, and environment dev.
