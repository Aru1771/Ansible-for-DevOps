Ansible Facts and Gathering Facts:
======================================

What are Ansible Facts?
------------------------

* Ansible Facts are system information automatically collected from remote servers.

For example, suppose you have three EC2 instances. You want to know:

    - Which Linux distribution is installed?
    - What is the private IP address?
    - How much RAM is available?
    - How many CPUs does the server have?
    - Which architecture is being used?
    - What is the hostname?

* You don't need to log in to every EC2 instance and run Linux commands manually.

* Ansible can collect this information automatically using Fact Gathering.

How does Ansible collect facts?
-------------------------------

* When you execute a playbook, Ansible normally gathers facts automatically before executing the tasks.

<img width="577" height="600" alt="image" src="https://github.com/user-attachments/assets/d616673d-1530-4452-909d-0a180fc51791" />

The module responsible for gathering facts is:

    ansible.builtin.setup

You generally don't need to call it manually because fact gathering is enabled by default.

Collect EC2 server information:
-------------------------------
Let's create a playbook called facts.yml.


    ---
    - name: Gather EC2 server facts
      hosts: dev
      become: true
      gather_facts: true
    
      tasks:
        - name: Display operating system
          ansible.builtin.debug:
            msg: "OS: {{ ansible_facts['distribution'] }}"
    
        - name: Display OS version
          ansible.builtin.debug:
            msg: "OS Version: {{ ansible_facts['distribution_version'] }}"
    
        - name: Display hostname
          ansible.builtin.debug:
            msg: "Hostname: {{ ansible_facts['hostname'] }}"
    
        - name: Display private IP
          ansible.builtin.debug:
            msg: "Private IP: {{ ansible_facts['default_ipv4']['address'] }}"
    
        - name: Display total memory
          ansible.builtin.debug:
            msg: "Memory: {{ ansible_facts['memtotal_mb'] }} MB"
    
        - name: Display CPU count
          ansible.builtin.debug:
            msg: "CPU count: {{ ansible_facts['processor_vcpus'] }}"


Understand gather_facts: true

    gather_facts: true


* This tells Ansible to collect system information before executing the tasks.

* Actually, gather_facts is true by default, so writing it explicitly here is for learning purposes.

* Execute your playbook

      ansible-playbook -i inventory facts.yml


* Expected output will look similar to this (actual values depend on your EC2 instance):

      OS: Amazon
      OS Version: 2023
      Hostname: ip-172-31-32-87
      Private IP: 172.31.32.87
      Memory: 1900 MB
      CPU count: 2

Learn to collect facts without a playbook:
----------------------------------------------

* Ansible also lets you collect facts directly using an ad-hoc command.

Run:

    ansible dev -i inventory -m ansible.builtin.setup

* This displays a large amount of system information.

* You can filter specific information instead of displaying everything.

Only OS information:

    ansible dev -i inventory -m ansible.builtin.setup -a "filter=ansible_distribution*"

Only memory information:

    ansible dev -i inventory -m ansible.builtin.setup -a "filter=ansible_memtotal_mb"

Only network information:

    ansible dev -i inventory -m ansible.builtin.setup -a "filter=ansible_default_ipv4"

Real-world AWS DevOps use cases:
-------------------------------

* Here are three important production scenarios.

* This is very useful when troubleshooting EC2 instances.

<img width="995" height="805" alt="image" src="https://github.com/user-attachments/assets/3a810054-5eb2-4cd3-b045-b3dc4751f5b0" />

<img width="1022" height="265" alt="image" src="https://github.com/user-attachments/assets/f6132343-4ffe-457f-9297-4a87037be2a6" />

Disable fact gathering when unnecessary:
---------------------------------------
Sometimes you don't need system information.

For example:

    - name: Simple connectivity test
      hosts: dev
      gather_facts: false
    
      tasks:
        - name: Display message
          ansible.builtin.debug:
            msg: "EC2 connectivity successful"
    

Why use gather_facts: false?
- Reduces unnecessary fact-gathering work.
- Can make simple playbooks execute faster.
- Useful when tasks don't require system information.

Remember: if you disable fact gathering, don't expect the normal gathered facts to be available unless you collect them separately.

