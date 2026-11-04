Theory: What is variable precedence?

Variable precedence means:

* When the same variable is defined in multiple places, which value does Ansible use?

* Imagine your Empower application uses port 8080 by default, but your production EC2 server needs port 80.

You might define app_port in:

    - group_vars
    - host_vars
    - Playbook vars
    - Command-line extra variables (-e)
* If the same variable exists in multiple places, Ansible selects a value according to its precedence rules.

Understand with a simple example:
-----------------------------------

<img width="922" height="667" alt="image" src="https://github.com/user-attachments/assets/04fa8e4c-b11f-4b77-84c7-516ad45d4253" />

Practical task: Test variable precedence
Let's use your existing Ansible project.

Step 1 — group_vars/dev.yml

    app_port: 8080

Step 2 — host_vars/server01.yml

    app_port: 8085

Step 3 — precedence.yml

    ---
    - name: Understand variable precedence
      hosts: dev
    
      vars:
        app_port: 8090
    
      tasks:
        - name: Display application port
          ansible.builtin.debug:
            msg: "Application port is {{ app_port }}"
    
Run:

    ansible-playbook -i inventory precedence.yml


For server01, expected output:

    Application port is 8090
Why? Playbook-level vars takes precedence over the inventory variables in this example.

<img width="1007" height="866" alt="image" src="https://github.com/user-attachments/assets/5cde4155-f306-484b-8a88-2c3291f690ea" />

* This is particularly useful in CI/CD pipelines where Jenkins can pass a release-specific version, environment, or configuration value to Ansible.

* Note: This is a simplified learning order, not the complete official Ansible precedence table. Ansible has additional variable sources and rules.

Real-world AWS DevOps use case:
---------------------------------

Imagine Jenkins triggers an Ansible deployment:

    - Default application port: 8080
    - Development EC2 configuration: 8081
    - Individual server configuration: 8085
    - Jenkins passes an override: 8090
Ansible uses 8090 because Jenkins passes it as an extra variable.

Jenkins pipeline example:

    sh """
    ansible-playbook -i inventory deploy.yml \
    -e "app_version=FR7 environment=prod"
    """
This allows a pipeline to pass deployment-specific values without editing the playbook.

