## Instructions for deploying Splunk with Ansible

**Summary**
In this demo, I want to show you a quick and easy way to perform a bare installation of Splunk using Ansible. This is an Ansible project that installs or upgrades Splunk to a specific version. It can also perform basic OS config (ulimits, THP disabled, hostname, etc.), ./splunk/etc/ backups, and SSL cert installation. The main playbook that I had to modify was `roles/os-config/tasks/main.yml`.

NOTE: This demo did not originate with me. I created my own version of the following example: https://github.com/johnmcgovern/ansible-splunk-base.git

### Setup Instructions

1. A single Ansible Automation Controller is sufficient for this demo.

2. First, feel free to git clone this project:

~~~
git clone https://github.com/jbyrdrh/splunk-install-example.git
~~~
	
3. Two files need to modified for this demo. First, I'll change into the `ansible-splunk-base` directory.

~~~
cd ./ansible-splunk-base		
~~~

4. Copy the `hosts.sample` file to `hosts`

~~~
cp hosts.sample hosts
~~~

5. Edit the hosts file to include the desired managed nodes in your test environment.

~~~
vi hosts
~~~
	
6. Copy group_vars/all.sample to group_vars/all

~~~
cp group_vars/all.sample group_vars/all
~~~

7. Edit the `group_vars/all` variables as appropriate for your test environment.

~~~
vi group_vars/all
~~~

In this demo, I defined the following variables. The rest of the varibles can remain as the default values.:

~~~
ansible_user: ansible

splunk_user: <your_username>
splunk_pass: <your_password>

splunk_uf_user: <your_username>
splunk_uf_pass: <your_password>
~~~

### Usage
	
1. Navigate to playbook base directory

~~~
cd ./ansible-splunk-base
~~~
	
2. Run the Splunk install playbook

~~~
ansible-playbook -i hosts install.yml
~~~

*NOTE: I verfied that this playbook installs Splunk successfully on RHEL 9.*


3. -or- run the Splunk upgrade playbook

~~~
ansible-playbook -i hosts upgrade.yml
~~~

4. -or- run the Splunk OS initial configuration playbook (built to provide a simple configuration for lab hosts)

~~~
ansible-playbook -i hosts os-config.yml
~~~

In `roles/os-config/tasks/main.yml`, I modified the path variable for the location of the splunk forwarder rpm file.

5. -or- run a base OS config AND install Splunk.

~~~
ansible-playbook -i hosts combo.yml	
~~~

6. -or- configure an TLS/SSL key pair for the web UI (tcp/8000).

~~~
ansible-playbook -i hosts tls-config.yml						
~~~

7. -or- run the Splunk UF install playbook

~~~
ansible-playbook -i hosts uf-install.yml
~~~

8. -or- run the Splunk UF config playbook

~~~
ansible-playbook -i hosts uf-config.yml
~~~

9. -or- run the Splunk UF install AND config playbook

~~~
ansible-playbook -i hosts uf-combo.yml	
~~~

10. -or- run the Splunk UF upgrade playbook

~~~
ansible-playbook -i hosts uf-upgrade.yml					
~~~

11. -or- run the Splunk configuration only (./etc/) backup playbook

~~~
ansible-playbook -i hosts backup-etc.yml
~~~

12. -or- run the Splunk full backup (/opt/splunk/) playbook

~~~
ansible-playbook -i hosts backup-full.yml		
~~~

13. Run an Ansible playbook limited to certain hosts within the hosts list

~~~
ansible-playbook -i hosts --limit=host1 install.yml
~~~

14. Run multiple roles in one command

~~~
ansible-playbook -i hosts os-config install.yml tls-config.yml
~~~

### Compatibility

This role has been tested on:

- Ubuntu 22.04, 20.04, & 18.04 Server (LTS)
- RHEL 8/9
- CentOS 7 1810
- Amazon Linux 2 2022.06 & 2020.04


### Notes

- The goal of this role is to quickly execute a best-practices base Splunk install/upgrade (including support for Workload Management, which is a departure from the previous install method).
- There are more complex/full-featured projects out there for various deployment topologies. The goal here is simplicity, speed, and utility.
- 8.1.1 introduced PolicyKit (polkit) management of systemd processes which allows for splunk to be restarted (for example) as the splunk user or super user using the commnands "splunk restart", "systemctl restart Splunkd", and "sudo systemctl restart Splunkd" for maximum flexibility.
- Both "systemd" and "initd" methods of Linux process management are supported. systemd is ONLY available in Splunk Enterprise version 7.2.2 and later. 
- Splunk versions 7.2.2 - 7.2.x implement "enable boot-start" differently than 7.3.0 and later. This is now accounted for.
- Assuming a semi-default install (such as you would find if you installed with this playbook), upgrade.yml will convert from initd process management to systemd process management if you flag "systemd" on install_method.
- A number of config items are set which disable pop-ups and modal dialogues which would normally be shown to the Splunk admin and/or users such as new version available notifications, UI tours, and python 2.7 deprication notifications. The goal here is to generally avoid UI annoyances that would crop up in automatic distributed Splunk deployments.
- This Ansible playbook does not currently handle OS-level firewall allowances for splunkd TCP ports.
- We bias towards being non-destructive. For example, if we see an existing/previous Splunk install we will fail out rather than damage the current install. 

### To-Do

- Support for additional server settings.
- Simplified version/file/hash dictionary.



