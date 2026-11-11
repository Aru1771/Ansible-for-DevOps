Conditions in Ansible:
=======================

Theory: What are conditions?

    Conditions allow Ansible to execute a task only when a particular condition is satisfied.

* In production DevOps, you don't always want to execute every task on every EC2 instance.

For example:

      - Install Apache only on Amazon Linux servers.
      - Install a different package on Ubuntu servers.
      - Restart Nginx only when its configuration changes.
      - Stop deployment if a required condition fails.
Ansible uses the when keyword for conditional execution.

Understanding when

Example:

    ---
    - name: Conditional execution
      hosts: dev
      become: true
      gather_facts: true
    
      tasks:
        - name: Install nginx on RedHat systems
          ansible.builtin.dnf:
            name: nginx
            state: present
          when: ansible_facts['os_family'] == "RedHat"

Understand the condition:

when: ansible_facts['os_family'] == "RedHat"


    - ansible_facts['os_family'] retrieves the OS family.
    - == compares two values.
    - "RedHat" is the condition we want to match.
    - If true, Ansible executes the task.
    - If false, Ansible skips the task.
Amazon Linux belongs to the RedHat OS family, so this condition matches Amazon Linux.


More real-world examples

Example A — Install packages based on OS

    tasks:
      - name: Install nginx on Amazon Linux
        ansible.builtin.dnf:
          name: nginx
          state: present
        when: ansible_facts['os_family'] == "RedHat"
    
      - name: Install nginx on Ubuntu
        ansible.builtin.apt:
          name: nginx
          state: present
        when: ansible_facts['os_family'] == "Debian"
    
Ansible chooses the appropriate task based on the target server's OS family.

Example B — Execute only in production

    - name: Restart production application
      ansible.builtin.debug:
        msg: "Production deployment started"
      when: environment == "prod"

Important: Don't put {{ }} around variables in a when condition.

Correct:

    when: environment == "prod"

Incorrect:

    when: "{{ environment == 'prod' }}"

Your practical exercise

* Create conditions.yml using your existing EC2 inventory.

Requirements:

    1. Enable fact gathering.
    2. Display the OS distribution.
    3. Install tree only when the OS family is RedHat.
    4. Display Amazon Linux detected only when the distribution is Amazon.
Expected conditional logic:

    when: ansible_facts['os_family'] == "RedHat"


    when: ansible_facts['distribution'] == "Amazon"


Run:

    ansible-playbook -i inventory conditions.yml

What we'll cover in this Conditions module
Concept	Purpose
when	Execute tasks conditionally
register + when	Execute tasks based on previous task output
changed_when	Control when Ansible reports a task as changed
failed_when	Define custom failure conditions

Your task now: Write conditions.yml and share your playbook. I'll review it line by line.


      
