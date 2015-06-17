class: center, middle, inverse

# Infrastructure Testing with Ansible
.footnote[[ThoughtWorks & Suncorp]()]

---
class: center, middle, inverse
# Can we test infrastructure?
![Default-aligned image](ansible/images/Robots.jpg)
<!-- yes, of course. ops always manuually test. varification scripts maybe? can we know if 
the command/instructruction we type in prod enviroment will work not bring down catastrophically any server?-->

---
class: center, middle, inverse
# Can't we just always manually test everything in production?
![Default-aligned image](ansible/images/test_in_prod.gif)

---
class: center, middle, inverse
# Can we trust human testers?
![Default-aligned image](ansible/images/disaster-girl.jpg)
<!-- 
A tester will decide what to test, and what not. He will maybe make mistakes during testing. All these problems can be prevented with automated tests
-->

---
class: center, middle, inverse
# What about testing prior to production?
# What about automated testing for infrastructure? 
![Default-aligned image](ansible/images/automate_all.png)

---
class: center, middle, inverse
# Infrastructure as code 
# makes it possible!
![Default-aligned image](ansible/images/infra_as_code.png)


---
class: center, middle, inverse
# What does it really mean?

<!-- 
Infrastructure as code, or programmable infrastructure, means writing code (which can be done using a high level language or any descriptive language) to manage configurations and automate provisioning of infrastructure in addition to deployments. This is not simply writing scripts, but involves using tested and proven software development practices that are already being used in application development. For example: version control, testing, small deployments, use of design patterns etc. In short, this means you write code to provision and manage your server, in addition to automating processes.
-->

---
class: center, middle, inverse
# It means
## .red[version control]
## .red[testing]
## .red[small deployments]
## use of design patterns
## ...

---
class: center, middle, inverse
# Testing infrastructure with Ansible
![Default-aligned image](ansible/images/Ansible_as_Glue.png)

---
class: center, middle, inverse
# What is Ansible?
## An ansible is a fictional machine capable of sending and receiving messages to and from a corresponding device over any distance whatsoever with no delay. Ansibles occur as plot devices in science fiction literature.

---
class: center, middle, inverse
# What is Ansible?
.left[
## It is a platform for configuring and managing computers
## Architecture concept:
## - controlling machines
## - nodes
]
<!--
As with most configuration management software, Ansible distinguishes two types of servers: controlling machines and nodes. First, there is a single controlling machine which is where orchestration begins. Nodes are managed by a controlling machine over SSH. The controlling machine describes the location of nodes through its inventory.
  -->

---
class: center, middle, inverse
# Inventory of Ansible
## a description of the nodes that can be accessed by Ansible
## described by a configuration file

---
class: center, middle, inverse
# Playbooks of Ansible
## configurations
## deployment
## orchestration

---
class: center, middle, inverse
# Playbooks of Ansible

.left[
```ruby
minna.yml

- hosts: webservers
  remote_user: root
  
  tasks:
    - name: test connection
      ping:

```
]

---
class: center, middle, inverse
# Run an Ansible playbook
.left[
```ruby

$ ansible-playbook -i inventory minna.yml

PLAY [tag_Purpose_cc_self_service_testing] ************************************

GATHERING FACTS ***************************************************************
ok: [10.181.51.32]

TASK: [test connection] *******************************************************
ok: [10.181.51.32]

PLAY RECAP ********************************************************************
10.181.51.32               : ok=2    changed=0    unreachable=0    failed=0

```
]

---
class: center, middle, inverse
# Now let's talk about testing with Ansible

---
class: center, middle, inverse
# "Test driven" provision/deployment
<!-- There are many options. Ansible is actually designed to be a “fail-fast” and ordered system, therefore it makes it easy to embed testing directly in Ansible playbooks.

By incorporating a degree of testing into your deployment workflow, there will be fewer surprises when code hits production and, in many cases, tests can be leveraged in production to prevent failed updates from migrating across an entire installation. Since it’s push-based, it’s also very easy to run the steps on the localhost or testing servers. Ansible lets you insert as many checks and balances into your upgrade workflow as you would like to have.

-->
---
class: center, middle, inverse
# “fail-fast” 
## embed testing directly in Ansible playbooks

---
class: center, middle, inverse
# Rolling Updates pattern
![Default-aligned image](ansible/images/rolling_update.jpg)

---
class: center, middle, inverse

#You can use success or failure of the playbook run to decide whether to add a machine into a load balancer or not
.left[
```ruby
- hosts: webservers

  pre_tasks:

    - name: take out of load balancer pool
      command: /usr/bin/take_out_of_pool {{ inventory_hostname }}
      delegate_to: 127.0.0.1

  roles:

     - common
     - webserver

  tasks:
*     - script: /srv/qa_team/app_testing_script.sh --server {{ inventory_hostname }}
*       delegate_to: testing_server

  post_tasks:

    - name: add back to load balancer pool
      command: /usr/bin/add_back_to_pool {{ inventory_hostname }}
      delegate_to: 127.0.0.1
```
]

<!--
  In the above example, a script is run from the testing server against a remote node prior to bringing it back into the pool. what you can see from the above is that tests are used as a gate – if the “apply_testing_checks” step is not performed, the machine will not go back into the pool.

In the event of a problem, fix the few servers that fail using Ansible’s automatically generated retry file to repeat the deploy on just those servers.
  -->

---
class: center, middle, inverse
# basic validation of your application in your playbooks will run .red[every time] you deploy

---
class: center, middle, inverse
# Modules That Are Useful for Testing
.left[
```ruby

*- wait_for: host={{ inventory_hostname }} port=22
*- action: uri url=http://www.example.com return_content=yes
*- script: test_script2 --parameter value --parameter2 value
*- assert:
    that:
      - "'error' not in cmd_result.stderr"
*- stat: path=/path/to/something
```
]

---
class: center, middle, inverse
# Absolutely .red[No Need] to check return codes of commands 
## .left[Ansible is checking them automatically]
### .left[e.g., rather than checking if a user exists, consider the user module to make it exist]

---
class: center, middle, inverse
# What to test?
## One must keep satisfying popular requirements when thinking of test automation

---
class: center, middle, inverse
# What to test?
## For BT Claims Platform team, our goal is to provide:
## .red[Stable and reliable infrastructure]

---
class: center, middle, inverse
# What to test?
.left[
# .red[Inwards:]
## our infrastructure relies on:
## - external teams (AMI, bootstrap script...)
## - external systems

# .red[Outwards:]
## - self services
]

---
class: center, middle, inverse
# What to test?
.left[
## - Tests for ensuring successful creation and termination of EC2 instances
## - Tests for ensuring reliable self services
]

---
class: center, middle, inverse
# Testing Lifecycle
.left[
```ruby

- Use the same playbook all the time with embedded tests in development
- Use the playbook to deploy to a test environment (with the same playbooks) 
that simulates production
- Run integration/acceptance tests against test environments
- Deploy to production

```
]

---
class: center, middle, inverse
# Integration/Acceptance tests?
.left[
```ruby
- Selenium tests 
- automated API tests 
- anything that is not embedded into your Ansible playbooks!
```
]

---
class: center, middle, inverse
# Continuous Integration
![Default-aligned image](ansible/images/guidewire-ci-screenshot.JPG)

---
class: center, middle, inverse
# Achieving Continuous Deployment
.left[
```ruby
- Write and use automation to deploy local development VMs or any other VMs/hosts 
for testing
- Have a CI system like Jenkins deploy to a stage environment on every code change
- The deploy job calls testing scripts to pass/fail a build on every deploy
- If the deploy job succeeds, it runs the same deploy playbook against production 
inventory
```
]	

---
class: center, middle, inverse
# Wow, .red[Continuous Deployment]
# Can we do it?
![Default-aligned image](ansible/images/cd.gif)

---
class: center, middle, inverse
# What does it mean to us?

---
class: center, middle, inverse
# It means
## .red[automated provisioning]
## .red[automated tests]
## .red[fail safe environments]
## .red[repeatable everything]

---
class: center, middle, inverse
# What do we have right now?
## some automated provision/deployment
## a little bit of automated tests
## fail safe environments ... eh?
## almost repeatable everything

---
class: center, middle, inverse
# What to do from here?

---
class: center, middle, inverse
## .left[continue to grow and enrich test automation suite]
## .left[build pipelines for different intentions]
## .left[make everything visible!]

---
class: center, middle, inverse
# not the end!
![Default-aligned image](ansible/images/panda.gif)

---
class: center, middle, inverse
![Default-aligned image](ansible/images/DevOps-with-Puppet-Chef-Ansible.jpg)

---
class: center, middle, inverse
![Default-aligned image](ansible/images/calms-model.png)

