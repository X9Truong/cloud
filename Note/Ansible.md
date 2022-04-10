### Ansible 2022

### 1. A Playbook with Ansible Find – to find the Directories alone

```
---
- name: Ansible Find Example
  hosts: testserver
  vars:
    Files: []
  tasks:
   - name : Find files bigger than 100mb in size
     become: true
     find:
       paths: /var/log
       file_type: directory
       recurse: no
       excludes: 'nginx,mysql'
     register: output

   - name: Adding Files to the LIST
     no_log: true
     set_fact:
       Files: "{{ Files + [item.path]}}"
     with_items: "{{ output.files }}"

   - debug: var=Files
```   

*- In the playbook, You could see that we are having a three tasks. The first one is to find the directories. It is achieved by setting  file_type: directory we are having no recursive as we do not want to traverse the subdirectories. In other words, we want only the directories of `/var/log`*

*- We are excluding the directories named `nginx` and `mysql` from the result.*

*- In the next task. we are traversing through the output register variable captured in the previous task and just filtering the directory names alone and creating an array (or) list using `set_fact`*


```
---
- name: Ansible Find Example
  hosts: testserver
  tasks:
   - name : Find files bigger than 100mb in size
     become: true
     find:
       paths: /
       file_type: file
       size: 100m
       recurse: yes
     register: output

   - debug: var=item.path
     with_items: "{{ output.files }}"
```	 


### 2. Check md5sum and compare

```
---
- hosts: mysql
  gather_facts: no
  tasks:
    - block:
      - name: get properties 
        shell: md5sum /etc/ansible/check_network.yml
        delegate_to: localhost        
        register: src_info        
      - name: Debug md5sum
        debug:
          var: src_info.stdout.split(' ')[0]    
      - name: copy src.txt to dest.txt
        copy:
          src: /etc/ansible/check_network.yml
          dest: /tmp
          force: yes
      - name: get properties  2
        shell: md5sum /tmp/check_network.yml
        register: copy_out        
      - name: Debug md5sum 2
        debug:
          var: copy_out.stdout_lines           
      - name: Fail if copy was a failure
        fail:
          msg: "Copy failed!"
        when: src_info.stdout.split(' ')[0] != copy_out.stdout.split(' ')[0]
      - name: Print Copy successful
        debug:
          msg: "Copy Successful!"
        when: src_info.stdout.split(' ')[0] == copy_out.stdout.split(' ')[0]
```		

### 3. A Playbook with Ansible Find

```
---
- name: Ansible Find Example
  hosts: testserver
  vars:
    Files: []
  tasks:
   - name : Find files bigger than 100mb in size
     become: true
     find:
       paths: /var/log
       file_type: file
       patterns:  '^[a-z]*_[0-9]{8}\.log$'
       size: 100m
       use_regex: yes
     register: output

   - name: Adding Files to the LIST
     no_log: true
     set_fact:
       Files: "{{ Files + [item.path]}}" 
     with_items: "{{ output.files }}"

   - debug: var=Files
```   

```
100m – Would find the files grater than or equal to 100mb size

-100m – Would find the smaller files with 100mb or below in size
```

```
---
- name: Ansible Find Example
  hosts: testserver
  vars:
    Files: []
  tasks:
   - name : Find files bigger than 100mb in size
     become: true
     find:
       paths: /var/log
       file_type: file
       patterns: 
         - '^[a-z]*_[0-9]{8}\.log$'
         - '^_[0-9]{2,4}_.*.log$'
         - '^[a-z]{1,5}_.*log$'
       size: 100m
       use_regex: yes
     register: output

   - name: Adding Files to the LIST
     no_log: true
     set_fact:
       Files: "{{ Files + [item.path]}}" 
     with_items: "{{ output.files }}"

   - debug: var=Files
```   

```
---
- name: Ansible Find Example
  hosts: testserver
  tasks:
   - name : Find files older than 30 days
     become: yes
     find:
       paths: /var/log
       patterns: '*.log'
       age: 30d
       age_stamp: mtime
     register: output

   - name: Delete the files matching
     become: yes
     file:
       path: "{{item.path}}"
       state: absent
     with_items: "{{ output.files }}"
```	 

### 4. AD-HOC Command

```
ansible mysql -m command -a uptime -i hosts.yml
ansible mysql -m command -a "df -h" -i hosts.yml
```


### 5. Check hosts

```
ansible-inventory -i hosts.yml --list
ansible -i hosts.yml -m debug -a "var=hostvars[inventory_hostname].mode" all		

{% for host in groups['all'] %}
{{ hostvars['host']['ansible_facts']['default_ipv4']['address'] }}
{% endfor %} 
```

### 6. Ansible Unarchive Module Examples

```
Some Quick notes before we see an example of this module

- This module requires zipinfo and gtar/unzip command on the target remote host.
- Can handle .zip files using unzip as well as .tar, .tar.gz, .tar.bz2 and .tar.xz files using gtar.
- Does not handle .gz files, .bz2 files or .xz files that do not contain a .tar archive.
- It Uses gtar’s --diff arg to calculate if changed or not. If this arg is not supported, it will always unpack the archive.
- Unarchive could do it as a Single task as it can optionally copy the file from control machine to target remote server and uncompress it there.
```

```
---
- name: Playbook to copy file and uncompress
  hosts: appservers
  vars:
    - userid : "weblogic"
    - oracle_home: "/opt/oracle"
    - jdk_instl_file: "server-jre-8u191-linux-x64.tar.gz"

  tasks:
  - name : Copy and Install Java
    become: yes
    become_user: "{{ userid }}"
    unarchive:
      src: "{{ item }}"
      dest: "{{ oracle_home }}"
      mode: 0755
    with_items:
      - "{{ jdk_instl_file }}"
```	  
```
---
- name: Playbook to download and install tomcat8
  hosts: appservers

  tasks:
  - name: install Java
    become: yes
    yum:
      name: java-1.8.0-openjdk-devel
      state: present

  - name: crate a directory
    become: yes
    file:
      path: "/opt/tomcat8"
      state: directory
      mode: 0755

  - name : Download and install tomcat
    become: yes
    tags: installtc
    unarchive:
      src: "https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.49/bin/apache-tomcat-8.5.49.tar.gz"
      dest: "/opt/tomcat8/"
      mode: 0755
      remote_src: yes
    register: "tcinstall"

  - name: Start the tomcat instance
    become: yes
    shell:
      "./startup.sh"
    args:
      chdir: "/opt/tomcat8/apache-tomcat-8.5.49/bin"
```	  

### 7. Ansible List Examples – How to create and append items to List

<img src="/Note/img/as1.png">

```
---
- name: Ansible List Examples
  hosts: localhost
  tasks:
    - name: Create a List variable and print it
      set_fact: 
        Continents: ["Asia","Africa","Europe","North America","South America","Antarctica","Australia"]
        Countries : ['India','Japan', 'Norway', 'Netherlands', 'Switzerland', 'Germany', 'United States of America']
        
    - name: Print the Continents
      debug: var=Continents
    - name: Print the Countries
      debug: var=countries
```

```
---
- name: Ansible List Examples
  hosts: localhost
  vars:
    Continents: ["Asia","Africa","Europe","North America","South America","Antarctica","Australia"]
    Countries : ['India','Japan', 'Norway', 'Netherlands', 'Switzerland', 'Germany', 'United States of America']
  tasks:
    - name: Print the Continents
      debug: var=Continents
    - name: Print the Countries
      debug: var=countries
```

* How to append new elements to Ansible List – Append	  

* Here is the Ansible playbook, to append elements to the list.
	
```
---
- name: Ansible List Examples
  hosts: localhost
  tasks:
    - name: Create a List variable and print it
      set_fact: 
        Countries : ['India','Japan', 'Norway', 'Netherlands', 'Switzerland', 'Germany', 'United States of America']
    - name: Print the Countries before adding element
      debug: var=Countries
    - name: Add new element to Countries
      set_fact: 
        Countries: "{{ Countries + ['United Kingdom', 'Russia'] }}"
    - name: Print the Countries after adding element
      debug: var=Countries
```	

* We are adding two countries `UnitedKingdom` and `Russia` to the list

`Countries: "{{ Countries + ['United Kingdom', 'Russia'] }}"`

* What we are basically doing here is a merge of two lists.  Something like this

```
NewCountries = ['United Kingdom', 'Russia']
Countries = Countries + NewCountries
```

```
---
- name: Ansible List Examples
  hosts: localhost
  tasks:
    - name: Create a List variable and print it
      set_fact:
        UserRecords: []
    - name: Print the userRecords
      debug: var=UserRecords
    - name: Add new user Records
      set_fact:
        UserRecords: '{{ UserRecords + [ {"name" : "Sarav", "MobileNo" : "9876543210" }, {"name" : "Hanu", "MobileNo" : "9879896210" } ] }}'
    - name: Print the userRecords after adding element
      debug: var=UserRecords
```	  

How to append Numbers to an ansible List

```
---
- name: Ansible List Examples
  hosts: localhost
  tasks:
    - name: Create a List variable and print it
      set_fact:
        Numbers: []
    - name: Print the Numbers
      debug: var=UserRecords
    - name: Add new Numbers to Ansible list
      set_fact:
        Numbers: '{{ Numbers + [ 1, 2, 3, 4] }}'
    - name: Print the Numbers after appending
      debug: var=Numbers
    - name: find the max value
      debug: 
        msg: "Biggest Number in this list is : {{ Numbers | max }}"
    - name: find the min value
      debug: 
        msg: "Smallest Number in this list is : {{ Numbers | min }}"
```




### 8.

