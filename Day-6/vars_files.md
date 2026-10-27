Topic 4: vars_files
====================

Theory: What is vars_files?

* vars_files allows you to store variables in a separate YAML file and load them into your playbook.

* Why is this useful?
  Imagine your Empower application has many settings: application name, release version, environment, port, installation directory, and service name.
  Keeping all these values directly inside the playbook can make it difficult to maintain.

* Instead, you can separate the configuration from the tasks.

<img width="982" height="557" alt="image" src="https://github.com/user-attachments/assets/8eb354c0-da5e-4068-8f79-b9fa36ddd3b9" />


The key difference from what you've already learned:

    - vars: variables are defined directly inside the playbook.
    - group_vars: variables are automatically loaded for an inventory group.
    - host_vars: variables are automatically loaded for an individual inventory host.
    - vars_files: the playbook explicitly names the file containing the variables it needs.

Practical task: Load variables from a separate file
----------------------------------------------------

* We'll create two files in your existing ansible-lab project.

Step 1 — Create the directory

    mkdir -p vars


* Your directory will look like this:

      ansible-lab/
      ├── inventory
      ├── deploy.yml
      ├── vars/
      │   └── dev.yml
      ├── group_vars/
      └── host_vars/


Step 2 — Create vars/dev.yml

* Put the application configuration in this file:

      app:
        app_name: Empower
        app_version: "FR6"
        environment: dev
        port: 8081
        directory: /opt/Empower

* Notice that this file contains only variables. It has no hosts, tasks, or vars_files sections.

Step 3 — Create deploy.yml

    ---
    - name: Load Empower application configuration
      hosts: dev
    
      vars_files:
        - vars/dev.yml
    
      tasks:
        - name: Display application name
          ansible.builtin.debug:
            msg: "Application: {{ app.app_name }}"
    
        - name: Display application version
          ansible.builtin.debug:
            msg: "Version: {{ app.app_version }}"
    
        - name: Display environment and port
          ansible.builtin.debug:
            msg: "Environment: {{ app.environment }}, Port: {{ app.port }}"
    
        - name: Display application directory
          ansible.builtin.debug:
            msg: "Directory: {{ app.directory }}"



Let's understand the important part:

    vars_files:
      - vars/dev.yml


* This tells Ansible to load the variables from vars/dev.yml. The playbook can then access them using expressions such as {{ app.app_name }}.

Step 4 — Execute the playbook

First, validate the syntax:

    ansible-playbook -i inventory deploy.yml --syntax-check


Then run it:

    ansible-playbook -i inventory deploy.yml


Expected output includes:

    Application: Empower
    Version: FR6
    Environment: dev, Port: 8081
    Directory: /opt/Empower

* This task only displays configuration; it does not deploy the application or modify your EC2 server.

3. Real-world AWS DevOps use case

* Suppose you maintain separate configurations for development and production:
      
      vars/
      ├── dev.yml
      └── prod.yml


* Your development file could contain port 8081, while production uses port 80. You can load the appropriate file with vars_files in the playbook.

* Be careful to select the correct production configuration when deploying; a wrong environment file can cause an incorrect deployment.

4. Your exercise

Now practise it yourself. Create vars/dev.yml with these values:
- Application name: Empower
- Application version: FR7
- Environment: dev
- Port: 8083
- Directory: /opt/Empower
Then create a playbook that loads this file using vars_files and displays the application version, environment, and port.

Your task: Send me your vars/dev.yml and the relevant part of your playbook. I'll review them line by line.
