Variables using vars
=====================

What is variables:
------------------

You have already learned this approach. Now let's understand when it is useful in real projects.

Theory:

The vars section defines variables inside a playbook. These variables are available to the tasks in that play.

Example:

    ---
    - name: Deploy Empower application
      hosts: dev
      become: true
    
      vars:
        app_name: Empower
        app_version: "FR4"
        environment: dev
        app_port: 8080
    
      tasks:
        - name: Display deployment details
          ansible.builtin.debug:
            msg: "Deploying {{ app_name }} version {{ app_version }} to {{ environment }} on port {{ app_port }}"


Theory: What is group_vars
---------------------------

group_vars lets you define variables for an entire inventory group. Instead of repeating variables in every playbook, you keep shared settings in a separate YAML file.
For example, suppose you have three development EC2 instances:

    - dev-server01
    - dev-server02
    - dev-server03
All three need the same application environment and port. You can define those values once in group_vars/dev.yml.

2. Practical task

Assume your project directory looks like this:

    ansible-lab/
    ├── inventory
    ├── deploy.yml
    └── group_vars/
        └── dev.yml


Step 1 — Create the directory

    mkdir -p group_vars


Step 2 — Create group_vars/dev.yml

    app:
      app_name: Empower
      app_version: "FR5"
      environment: dev
      port: 8080
      directory: /opt/Empower



Step 3 — Create deploy.yml

    ---
    - name: Deploy Empower application
      hosts: dev
      become: true
    
      tasks:
        - name: Display application configuration
          ansible.builtin.debug:
            msg: >-
              Application={{ app.app_name }},
              Version={{ app.app_version }},
              Environment={{ app.environment }},
              Port={{ app.port }},
              Directory={{ app.directory }}



Notice that deploy.yml contains no vars: section. 
Ansible loads group_vars/dev.yml automatically when the inventory group is named dev and the directory is in the expected location relative to the playbook or inventory.

Step 4 — Run the playbook

    ansible-playbook -i inventory deploy.yml


3. Real-world AWS DevOps use case

You can use separate group variable files for different environments:
File	Example settings

    group_vars/dev.yml	Port 8080, environment dev
    group_vars/prod.yml	Port 80, environment prod

This helps reuse one deployment playbook across development and production EC2 instances.

Note:

    the group_vars filename should match the inventory group name when you want Ansible to automatically load variables for that group.
