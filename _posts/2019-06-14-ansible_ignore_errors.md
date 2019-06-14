---
layout: post
title: Ansible ignore_errors
comment_id: 5
categories: [tech, ansible]
tags: [tech, ansible]
---

There are times that we do need to build some error handling logic into our playbooks. Ansible provides some built in directives and modules to help. In this post we are going to focus on `ignore_errors`. Ignore errors is not an excuse to ignore logic. If you find yourself using this often, you may want to step back and make sure that you're no drifting away from the idempotent philosophy of Ansible. Take this example below, 
```
---
- hosts: all
  gather_facts: no

  tasks:
  - name: check for port 80
    wait_for:
      port: 80
      timeout: 5
    register: check_web

  - debug:
      msg: “Port 80 is not open and listening”
    when: check_web.failed
```

What will happen when we run this playbook? Will we ever see the debug message? No, we won't. If port 80 is open, the conditional will cause the debug to be skipped. If port 80 is closed, the playbook will simply fail on the first task and we will never see the debug message even though it seems that is the intent of the developer. This is because Ansible's default behavior is stop execution when there is a failure. If all this playbook did was to check port 80, having the playbook fail on that task might get our point across. What if we are running a series of compliance checks? How about if we want a Slack notification or a ServiceNow incident opened if this port is not responding? Having the playbook fail on that task will not do a whole lot of good. Let's modify this playbook slightly.

```
---
- hosts: all
  gather_facts: no

  tasks:
  - name: check for port 80
    wait_for:
      port: 80
      timeout: 5
    ignore_errros: yes
    register: check_web

  - debug:
      msg: “Port 80 is not open and listening”
    when: check_web.failed
```

On the first task, `wait_for`, you will notice the line `ignore_errors: yes` has been added. This will prevent Ansible from failing the entire playbook on that task. This will proceed to the debug task where our `when` statement will check for the failure of the first module and display the debug accordingly. There are many more relevant things to do other than just debug a message here as I alluded to above such as integrating with Slack or ServiceNow. Maybe take shortcuts by only including the web server role if there is not one running already. It could also be used to do more complicated conditional logic on a return code or API response. I would like to point out an example of when this should NOT be used. Take this playbook as an example.

```
---
- hosts: all
  gather_facts: no

  tasks:
  - name: check for port 80
    wait_for:
      port: 80
      timeout: 5
    ignore_errros: yes
    register: check_web

  - service:
      name: httpd
      state: started
    when: check_web.failed
```

This would be bad Ansible. Ansible is intended to be idempotent, so we are doing redundant work here. If my goal is to start the web server if it's not running, that logic is already built into the module. The service module will only start the service if it is not running. So I don't need to build in my own logic to check to port to decide if the service needs to be started.
