---
layout: post
title: Define your own Failure
comment_id: 6
categories: [tech, ansible]
tags: [tech, ansible]
---

Ansible determines success of a playbook by the success or failure of the individual tasks in the play. Sometimes this method is limiting because it does not account for higher level logic that might dictate a failure however, there is a module that can be used to account for this. The `fail` module allows Ansible to process a conditional statement and will fail the play if the condition fails. Lets look at an example. 

```
---
- hosts: all
  gather_facts: no

  tasks:
  - name: check for port 80
    wait_for:
      port: 80
      timeout: 5
    ignore_errors: yes
    register: check_http

  - name: check for port 443
    wait_for:
      port: 443
      timeout: 5
    ignore_errors: yes
    register: check_https

  - fail:
      msg: "Port 80 and 443 are not open and listening"
    when: check_https.failed and check_http.failed

```

In this example, we have have two tasks checking if a port 80 and 443 are open on a target host. We only want to fail if both ports are not open. To do this, we add `ignore_errors` to to each of our `wait_for` tasks. See the [ignore_errors]({% post_url 2019-06-14-ansible_ignore_errors %}) post for more information on this parameter. This will prevent Ansible from failing on that task. We also need to register the output of each `wait_for` task for later. After both of our tests, we can use the `fail` module to evaluate if the play is successful or not. The `msg` parameter allows us to log a custom message when this module runs and a simple when condition checks for our condition. See the complete documentation [here](https://docs.ansible.com/ansible/latest/modules/fail_module.html).

