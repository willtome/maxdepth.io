---
layout: post
title: Ansible failed_when
comment_id: 4
categories: [tech, ansible]
tags: [tech, ansible]
---

This begins a series of posts I will be publishing on error handling. The first will focus on the `failed_when` directive. This allows for you to define what constitutes a "failure" even though the module may define it differently. Here is an example...
```
---
- hosts: all
  gather_facts: no

  tasks:
  - name: return false
    command: /usr/bin/false
    register: usr_bin_false
    failed_when: usr_bin_false.rc != 1
```
When you run `/usr/bin/false` the expected return code is 1. A return code of 1 would cause the command module to fail. If you're expecting the command to return 1, we may only want to fail if we don't receive 1. Using `failed_when` lets us control the failure condition. By adding the line `failed_when: usr_bin_false.rc != 1` the module will only fail when the return code is not 1, which in our example, would be when an actual failure has occurred.