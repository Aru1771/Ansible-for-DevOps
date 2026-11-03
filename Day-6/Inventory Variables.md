Topic 5: Inventory Variables:
=============================

Theory

* You already know that an Ansible inventory defines your EC2 servers and groups.
* We can also define variables directly inside the inventory file.

For example:
inventory:

    [dev]
    server01 app_port=8081 app_env=dev
    server02 app_port=8082 app_env=dev

Here:
    
    - server01 has port 8081.
    - server02 has port 8082.
    - Both belong to the dev group.
These are called host variables in the inventory.

Practical task

Create this playbook:

    ---
    - name: Display inventory variables
      hosts: dev
    
      tasks:
        - name: Display server configuration
          ansible.builtin.debug:
            msg: "Server={{ inventory_hostname }}, Port={{ app_port }}, Environment={{ app_env }}"

Run:

    ansible-playbook -i inventory inventory-vars.yml

Expected output:

    Server=server01, Port=8081, Environment=dev
    Server=server02, Port=8082, Environment=dev

Real-world AWS DevOps use case:

Imagine you have multiple EC2 instances:

    - Application server 1 listens on port 8081.
    - Application server 2 listens on port 8082.
    - Both belong to the development environment.
Inventory variables let you define these server-specific values directly alongside the host entries.

Your exercise:

Create inventory entries for:

    - server01: port 8085, environment dev
    - server02: port 8086, environment dev
Write the playbook to display both servers' configuration.

Send me your inventory and playbook. I'll review them before we move to Topic 6: Ansible Variable Precedence.
