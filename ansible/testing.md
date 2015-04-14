class: center, middle, inverse

# ansible, infrastructure testing and CI
.footnote[[ThoughtWorks & Suncorp]()]

---
class: center, middle, inverse
# Can we test infrastructure?
<!-- yes, of course. ops always manuually test. varification scripts maybe? can we know if 
the command/instructruction we type in prod enviroment will work not bring down catastrophically any server?-->

---
class: center, middle, inverse
# Operations always manually test everything in production!
<!-- instructions, and procedures are in the head -->

---
class: center, middle, inverse
# What about automated testing? What about testing prior to production?
<!-- instructions, and procedures are in the head -->

---
class: center, middle, inverse
# Infrastructure as code makes it possible!
<!-- instructions, and procedures are in the head -->

---
class: center, middle, inverse
# What does it really mean?
<!-- instructions, and procedures are in the head -->

---
class: center, middle, inverse
# Testing infrastructure with Ansible
<!-- instructions, and procedures are in the head -->

---
class: center, middle, inverse
# Testing infrastructure with Ansible
<!-- instructions, and procedures are in the head -->

---
class: center, middle, inverse
# "Test driven" provision
<!-- There are many options. Ansible is actually designed to be a “fail-fast” and ordered system, therefore it makes it easy to embed testing directly in Ansible playbooks.

By incorporating a degree of testing into your deployment workflow, there will be fewer surprises when code hits production and, in many cases, tests can be leveraged in production to prevent failed updates from migrating across an entire installation. Since it’s push-based, it’s also very easy to run the steps on the localhost or testing servers. Ansible lets you insert as many checks and balances into your upgrade workflow as you would like to have.

-->
---
class: center, middle, inverse
# Check Mode As A Drift Test
If running a deployment playbook against an existing and live system

