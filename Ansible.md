# Ansible Reference

### ad-hoc commands
Pinging the server to see if its the connection is working all right

  1. here I'm running this command in the same directory as my inventory file
  2. specify the user name by -u
  3. give the path to the key being used
  
```bash
ansible all --key-file ~/Documents/Keys/id_primary -i inventory -u ubuntu -m ping
```
---
Command equivalent to running update on server

```bash
ansible all -m apt -a update_cache=true --become --ask-become-pass
```

### Ansible Playbook syntax

```yml
---

- name: <playbook_name>
  hosts: all
  become: yes               # To run tasks with sudo privilages
  tasks:

  - name: <description>
    ansible.builtin.apt:    # relevant package manager
      name: <package_name>
      state: present        # by default the state is present 

  - name: install apache2
    ansible.builtin.apt:
      name: apache2
      state: present
```


	  
	  
- ```state: present```	- Ensures that the package is installed on the target hosts.
  - ```absent```	- Ensures that the package is not installed on the target hosts.
  - ```latest```	- Ensures that the package is installed and, if a newer version is 	available, Ansible will update the package to the latest version.
  - ```purged```	- Ensures that the package is purged from the system, removing its configuration files as well.


## Reference 
```yml
---

  - name: Install apache httpd  (state=present is optional)
    ansible.builtin.apt:
      name: apache2
      state: present
  
  - name: Update repositories cache and install "foo" package
    ansible.builtin.apt:
      name: foo
      update_cache: yes
  
  - name: Remove "foo" package
    ansible.builtin.apt:
      name: foo
      state: absent
  
  - name: Install the package "foo"
    ansible.builtin.apt:
      name: foo
  
  - name: Install a list of packages
    ansible.builtin.apt:
      pkg:
      - foo
      - foo-tools
  
  - name: Install the version '1.00' of package "foo"
    ansible.builtin.apt:
      name: foo=1.00
  
  - name: Update the repository cache and update package "nginx" to latest version using default release squeeze-backport
    ansible.builtin.apt:
      name: nginx
      state: latest
      default_release: squeeze-backports
      update_cache: yes
  
  - name: Install the version '1.18.0' of package "nginx" and allow potential downgrades
    ansible.builtin.apt:
      name: nginx=1.18.0
      state: present
      allow_downgrade: yes
  
  - name: Install zfsutils-linux with ensuring conflicted packages (e.g. zfs-fuse) will not be removed.
    ansible.builtin.apt:
      name: zfsutils-linux
      state: latest
      fail_on_autoremove: yes
  
  - name: Install latest version of "openjdk-6-jdk" ignoring "install-recommends"
    ansible.builtin.apt:
      name: openjdk-6-jdk
      state: latest
      install_recommends: no
  
  - name: Update all packages to their latest version
    ansible.builtin.apt:
      name: "*"
      state: latest
  
  - name: Upgrade the OS (apt-get dist-upgrade)
    ansible.builtin.apt:
      upgrade: dist
  
  - name: Run the equivalent of "apt-get update" as a separate step
    ansible.builtin.apt:
      update_cache: yes
  
  - name: Only run "update_cache=yes" if the last one is more than 3600 seconds ago
    ansible.builtin.apt:
      update_cache: yes
      cache_valid_time: 3600
  
  - name: Pass options to dpkg on run
    ansible.builtin.apt:
      upgrade: dist
      update_cache: yes
      dpkg_options: 'force-confold,force-confdef'
  
  - name: Install a .deb package
    ansible.builtin.apt:
      deb: /tmp/mypackage.deb
  
  - name: Install the build dependencies for package "foo"
    ansible.builtin.apt:
      pkg: foo
      state: build-dep
  
  - name: Install a .deb package from the internet
    ansible.builtin.apt:
      deb: https://example.com/python-ppq_0.1-1_all.deb
  
  - name: Remove useless packages from the cache
    ansible.builtin.apt:
      autoclean: yes
  
  - name: Remove dependencies that are no longer required
    ansible.builtin.apt:
      autoremove: yes
  
  - name: Remove dependencies that are no longer required and purge their configuration files
    ansible.builtin.apt:
      autoremove: yes
      purge: true
  
  - name: Run the equivalent of "apt-get clean" as a separate step
    apt:
      clean: yes
```
