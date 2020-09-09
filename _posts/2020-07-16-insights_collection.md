---
layout: post
title: Manage Red Hat Enterprise Linux like a Boss with Red Hat Ansible Content Collection for Red Hat Insights
comment_id: 7
categories: [tech, ansible]
tags: [tech, ansible]
---

Running IT environments means facing many challenges at the same time: security, performance, availability and stability are critical for the successful operation of today’s data centers. IT managers and their teams of administrators, operators and architects are well advised to move from a reactive, “fire-fighting” mode to a proactive approach where systems are continuously scanned and improvements are applied before critical situations come up. [Red Hat Insights](https://www.redhat.com/en/technologies/management/insights) routinely analyzes Red Hat Enterprise Linux systems for security/vulnerability, compliance, performance, availability and stability threats, and based on the results, can provide guidance on how to improve daily operations. Insights is included with your Red Hat Enterprise Linux subscription and located at cloud.redhat.com. 

We [recently announced](https://www.ansible.com/blog/now-available-the-new-ansible-content-collections-on-automation-hub) a new Red Hat Ansible Content Collection for Insights, an integration designed to make it easier for Insights users to manage Red Hat Enterprise Linux and to automate tasks on those systems using Ansible. The Ansible Content Collection for Insights is ideal for customers that have large Red Hat Enterprise Linux estates that require initial deployment and ongoing management of the Insights client.

In this blog, we will look at how this integration with Ansible takes care of key tasks via included Ansible Roles, Modules and Plugins, such as:

- Deploy the required packages to Red Hat Enterprise Linux instances on the cloud and on-premises 
- Register systems to Insights
- Provide custom facts to Ansible like the system ID, which can be reused in future  automation tasks

## Downloading the Collection
In order to add the Insights content to your playbooks, you can download them from either Automation Hub or Ansible Galaxy directly, or through the ansible-galaxy CLI tool. Automation Hub contains the certified content for Red Hat Ansible Automation Platform customers, while Ansible Galaxy contains the experimental community version.

To download the Collection, refer to Automation Hub (fully supported, requires an Red Hat Ansible Automation Platform subscription) or Ansible Galaxy (upstream community supported):

[Automation Hub Collection](https://cloud.redhat.com/ansible/automation-hub/redhat/insights): `redhat.insights`

[Ansible Galaxy Collection](https://galaxy.ansible.com/redhatinsights/insights): `redhatinsights.insights`

Information about how to configure downloading via the ansible.cfg file or requirements.yml file, please refer to the blog, “[Hands On With Ansible Collections](https://www.ansible.com/blog/hands-on-with-ansible-collections).”

## Deploying the Insights Client
Before you can start analyzing your systems with Insights, you must deploy the Insights Client to each server. Deploying any client or agent with Ansible makes the process painless. Even easier, integrate it into a provisioning workflow in Ansible Tower and you’ll never have to think about it again. A playbook to deploy the Insights Client with the provided role would look like the following: 
```
---
- name: deploy insights client to rhel servers
  hosts: rhel
 
  tasks:
  - include_role:
      name: redhat.insights.insights_client
```
By the way, for purposes of this blog, all examples will be using the fully qualified Collection name (FQCN) for content downloaded from Automation Hub.

It doesn’t stop there. The Insights Client can also report tags back to cloud.redhat.com to help you organize your servers. We’ll take a look at why this is really important in the next section, but for now, let’s focus on how to do it. Let’s say you want to indicate the environment, site, application group and default IP. Tags can be deployed by the Insights Client role by providing a variable called insights_tags. Let’s look at an example:
```
---
- name: deploy insights client with tags
  hosts: rhel
 
  tasks:
  - include_role:
      name: redhat.insights.insights_client
      vars:
        insights_tags:
          env: prod
          site: rdu
          app: ecomm
          default_ip: “{{ ansible_default_ipv4.address }}”
```
As you can see in the example, tags can be statically defined, computed from a fact or set using the value of another variable. If you are provisioning your server with Ansible Tower, you may have additional data about the cluster, vpc or network where the server is located. This type of data makes for great inputs for defining your Insights tags.

## Inventory: Who is What and What is Where
Now that you have the Insights Client deployed and properly tagged, you should have a nice consolidated list of your Red Hat Enterprise Linux servers on cloud.redhat.com. This list can be used as an Ansible inventory with the [Insights Inventory Plugin](https://cloud.redhat.com/ansible/automation-hub/redhat/insights/content/inventory/insights). If you are new to inventory plugins, do not confuse them with inventory scripts. While they seemingly produce the same result, they work very differently. To use the inventory plugin, we need to define an inventory file as a YAML file. In this inventory file, the different options for the plugin are specified. The most basic usage of the plugin will simply populate a list of all of your systems registered to Insights.
```
#insights.yml
---
plugin: redhat.insights.insights
```
There are a few very important things to note about what is happening in the background. First, the default Ansible configuration will automatically enable this plugin. Nothing additional needs to be set in a configuration file. The default [INVENTORY_ENABLED](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#inventory-enabled) setting includes a plugin called auto. Save yourself some unnecessary configuration and let auto do the work. The second very important detail is the name of the file; it must end with insights.yml. This is a design pattern for inventory plugins and you can prefix the file name with something like prod.insights.yml but the file name MUST end with insight.yml. Finally, to authenticate to Insights, set the environment variables `INSIGHTS_USER` and `INSIGHTS_PASSWORD` in your shell or via a [Custom Credential Type](https://docs.ansible.com/ansible-tower/latest/html/userguide/credential_types.html) in Ansible Tower. Push this file to your source control repository and then inventory source in Ansible Tower choosing “[Sourced from Project](https://docs.ansible.com/ansible-tower/latest/html/userguide/inventories.html#sourced-from-a-project)” as the type. Ansible Tower won’t show the file in the drop down, but you can type in the name of your file and click it to create the source. Next, let’s look at how to pull additional information into your inventory to make it more meaningful.

In the example inventory below, we use default variables to create additional variables and groups. Note: the inventory file name _must_ end in insights.yml. You can prefix this to differentiate like, prod.insights.yml if you’d like.
```
#insights.yml
---
plugin: redhat.insights.insights
get_patches: True
get_tags: True
compose:
  ansible_host: “{{insights_tags[‘insights_client’][‘default_ip’]}}”
groups:
  patching: insights_patching.enabled
  bug_patch: insights_patching.rhba_count > 0
  security_patch: insights_patching.rhsa_count > 0
  enhancement_patch: insights_patching.rhea_count > 0
keyed_groups:
  - key: insights_tags['insights-client'][‘env’]
    prefix: env
  - key: insights_tags['insights-client'][‘site’]
    prefix: site
  - key: insights_tags['insights-client'][‘app’]
    prefix: app
```
However, Insights knows much more about our systems and we can use that information to make our inventory more useful. The get_patches option will add host variables indicating the number of security, bug fix and enhancement patches the system is missing, and the get_tags option will allow creation of host variables based on the tags that we deployed with the `insights_client` role. 

This inventory file will create inventory groups for each category of patches as well as groups of each different environment, site and application group. If we need to apply security patches to all of the production servers running the ecomm application, [patterns](https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html) will select those hosts for you by setting the hosts line of your playbook to `env_prod:&app_ecomm`.

## Where to go from here?
In this blog, we walked through automating system registration with the Insights Client role followed by using cloud.redhat.com as an inventory source for running your Insights remediation playbooks and other general automation. More proactive management means less late-night outages, less failed upgrades, better security, better performance and more uninterrupted weekends. Red Hat Ansible Automation Platform and Insights put the right tools in your hands to proactively manage your environment at scale and couldn’t be simpler. 

Feel free to visit these other resources for more information:

- Insights Console at [cloud.redhat.com/insights (requires Red Hat subscription)](https://cloud.redhat.com/insights)
- [Getting started with Insights](https://access.redhat.com/products/red-hat-insights#getstarted)
- Learn more about Insights at [redhat.com/insights](http://redhat.com/insights)