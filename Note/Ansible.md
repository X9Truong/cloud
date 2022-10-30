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

* How to append Numbers to an ansible List

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




### 8.How to merge two Lists

```
---
- name: Ansible List Examples
  hosts: localhost
  tasks:
    - name: Create a Lists with Numbers, Strings and Dictionary
      set_fact:
        Numbers: [1,2,3,4]
        UserRecords: [ {"name" : "Sarav", "MobileNo" : "9876543210" }, {"name" : "Hanu", "MobileNo" : "9879896210" } ]
        Strings: ['test1','test2','test3']
        
    - name: Merging Ansible Lists
      debug: 
        msg: "{{ Numbers + Strings + UserRecords  }}"
```		


* How to access items from Ansible List

```
---
- name: Ansible List Examples
  hosts: localhost
  tasks:
    - name: Create a List variable and print it
      set_fact: 
        Countries : ['India','Japan', 'Norway', 'Netherlands', 'Switzerland', 'Germany', 'United States of America']
    - name: Print the Countries Ansible list
      debug: var=Countries
    - name: Print the first element using Index Number
      debug: 
        msg: "{{Countries[0]}}"
    
    - name: Print the first element using Ansible filter
      debug: 
        msg: "{{ Countries | first }}"
    
    - name: Print the third element using Index Number
      debug: 
        msg: "{{Countries[2]}}"
    - name: Print the last element using Ansible filter - last
      debug: 
        msg: "{{ Countries | last }}"
    - name: Print the last element using length and Index Number
      debug: 
        msg: "{{ Countries [Countries | length - 1 ] }}"
```		

* How to Apply Jinja2 Filters and Ansible Filters to List

```
---
- name: Ansible List Examples
  hosts: localhost
  tasks:
    - name: Create a Lists with Numbers, Strings and Dictionary
      set_fact:
        Numbers: [1,2,3,4]
        Numbers2: [4,5,6,7,8]
        NestedList: [1,2,[3,4],5,6]
        UserRecords: [ {"name" : "Sarav", "MobileNo" : "9876543210", "age": "32" }, {"name" : "Hanu", "MobileNo" : "9879896210", "age" : "31" } ]
        Strings: ['test1','test1','test3']
        
    - name: Ansible Min filter to find smallest number
      debug: 
        msg: "{{ Numbers | min }}"

    - name: Ansible Max filter to find Biggest number
      debug: 
        msg: "{{ Numbers | max }}"
    
    # To get the smallest value using Attribute on the list of dictionary
    # Finding the Youngest person
    - name: Filtering using Attribute and finding the Min value
      debug:
        msg: "{{ UserRecords | min(attribute='age') }}"
    
    # iterate through the list elements and get their power of 2!
    - name: Ansible pow filter to get the power of Number
      debug: 
        msg: "{{ item | pow(2) }}"
      with_items: 
        - "{{ Numbers }}"

    # iterate through the list elements and get their Square root
    - name: Ansible root filter to get the Square root
      debug: 
        msg: "{{ item | root }}"
      with_items: 
        - "{{ Numbers }}"

    # test1 duplicate be removed from the output
    - name: Ansible Get Unique value from the list
      debug:
        msg: "{{ Strings | unique }}"
    
    # Union of two lists, Duplicates would be removed
    - name: Union of two lists
      debug:
        msg: "{{ Numbers | union(Numbers2) }}"
      
    # Intersection of two lists ( finding common items present in both, unique)
    - name: Intersect of two lists
      debug:
        msg: "{{ Numbers | intersect(Numbers2) }}"

    # To get the difference of 2 lists (items in 1 that don’t exist in 2):
    - name: Intersect of two lists
      debug:
        msg: "{{ Numbers | difference(Numbers2) }}"

    # Flatten Nested List
    - name: Flatten the Nested Ansible List
      debug:
        msg: "{{ NestedList | flatten }}"
    
    # Shuffling the List
    - name: Shuffling the List to change the order
      debug:
        msg: 
          - "{{ NestedList | shuffle }}"
          - "{{ Strings | shuffle }}"
```

- Here is the output of this playbook

```
PLAY [Ansible List Examples] ****************************************************************************************

TASK [Gathering Facts] **********************************************************************************************
ok: [localhost]

TASK [Create a Lists with Numbers, Strings and Dictionary] **********************************************************
ok: [localhost]

TASK [Ansible Min filter to find smallest number] *******************************************************************
ok: [localhost] => {
    "msg": "1"
}

TASK [Ansible Max filter to find Biggest number] ********************************************************************
ok: [localhost] => {
    "msg": "4"
}

TASK [Filtering using Attribute and finding the Min value] ******************************************************************************
ok: [localhost] => {
    "msg": {
        "MobileNo": "9879896210",
        "age": "31",
        "name": "Hanu"
    }
}

TASK [Ansible pow filter to get the power of Number] ****************************************************************
ok: [localhost] => (item=1) => {
    "msg": "1.0"
}
ok: [localhost] => (item=2) => {
    "msg": "4.0"
}
ok: [localhost] => (item=3) => {
    "msg": "9.0"
}
ok: [localhost] => (item=4) => {
    "msg": "16.0"
}

TASK [Ansible root filter to get the Square root] *******************************************************************
ok: [localhost] => (item=1) => {
    "msg": "1.0"
}
ok: [localhost] => (item=2) => {
    "msg": "1.4142135623730951"
}
ok: [localhost] => (item=3) => {
    "msg": "1.7320508075688772"
}
ok: [localhost] => (item=4) => {
    "msg": "2.0"
}

TASK [Ansible Get Unique value from the list] ***********************************************************************
ok: [localhost] => {
    "msg": [
        "test1",
        "test3"
    ]
}

TASK [Union of two lists] *******************************************************************************************
ok: [localhost] => {
    "msg": [
        1,
        2,
        3,
        4,
        5,
        6,
        7,
        8
    ]
}

TASK [Intersect of two lists] ***************************************************************************************
ok: [localhost] => {
    "msg": [
        4
    ]
}

TASK [Intersect of two lists] ***************************************************************************************
ok: [localhost] => {
    "msg": [
        1,
        2,
        3
    ]
}

TASK [Flatten the Nested Ansible List] ******************************************************************************
ok: [localhost] => {
    "msg": [
        1,
        2,
        3,
        4,
        5,
        6
    ]
}

TASK [Shuffling the Ansible List to change the order] ***************************************************************
ok: [localhost] => {
    "msg": [
        [
            5,
            2,
            6,
            [
                3,
                4
            ],
            1
        ],
        [
            "test1",
            "test3",
            "test1"
        ]
    ]
}

PLAY RECAP **********************************************************************************************************
localhost                  : ok=13   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

* How to validate if the variable is a list


After learning various things about the Ansible list, we should also learn how to validate if the variable is in fact a list.

Ansible has a filter for the same `type_debug`

you can use it like this and it should result listif it is a valid list.

`{{ <the variable name> | type_debug }}`


* Ansible Map Examples – Filter List and Dictionaries | Devops Junction

Ansible Map Filter does two things

- It applies a filter on a sequence of objects
- It looks up an attribute.

This is useful when dealing with lists of objects but you are really only interested in a certain value of it.

The basic usage of map is to look up an attribute.  Let’s cover it first

Imagine you have a list of users but you are only interested in a list of usernames:

for example consider the following employee data with lot of values like first name, lastname, address, mobile etc.  But what I want is only their city


```
names: [ {
        "first": "Paul",
        "last": "Thompson",
        "mobile": "+1-234-31245543",
        "ctc": "100000",
        "address": {
          "city": "LasVegas",
          "country": "USA"
        }
      },
      {
        "first": "Sarav",
        "last": "AK",
        "mobile": "+919876543210",
        "ctc": "200000",
        "address": {
          "city": "Chennai",
          "country": "India"
        }

      }]
```

So this is how,  ansible map can help to list only the cities from the employee data.

`{{ names | map(attribute='address') | map(attribute='city')}}`

Or to get only the First Names

`{{ names | map(attribute='first') | join(',') }}` 

- Ansible Map Filter Examples – Looking up an Attribute

Here is the playbook with multiple Ansible Map filter examples with attribute selection

Each task has self-explanatory comments for you to understand what exactly they are doing.

```
---
- name: filter test
  hosts: localhost
  vars: 
    
    names: [ {
        "first": "Paul",
        "last": "Thompson",
        "mobile": "+1-234-31245543",
        "ctc": "100000",
        "address": {
          "city": "LasVegas",
          "country": "USA"
        }
      },
      {
        "first": "Rod",
        "last": "Johnson",
        "mobile": "+1-584-31551209",
        "ctc": "300000",
        "address": {
          "city": "Boston",
          "country": "USA"
        }
      },
      {
        "first": "Sarav",
        "last": "AK",
        "mobile": "+919876543210",
        "ctc": "200000",
        "address": {
          "city": "Chennai",
          "country": "India"
        }

      }]
  tasks:
  
  # Map Filter only selective attributes from list of objects [{},{}]
  - name: Select and Extract only the cities 
    debug: 
      msg="{{ names | map(attribute='address') | map(attribute='city')}}"

  # using attirubtes with list of objects [{},{}] - Selecting only mobile numbers
  - name: Select and Extract only mobile numbers
    debug:
      msg: "{{ names | map(attribute='mobile') }}"

  # Select Attributes Joined with Comma in Singleline ( By Default it returns a List)
  - debug: msg={{ names | map(attribute='first') | join(',') }} 
  - debug: msg={{ names | map(attribute='last') | join(',') }} 

  # Convert the lastname to uppercase
  - debug: msg={{ names | map(attribute='last') | map('upper') }} 

  # Convert the CTC attriute to float value
  - debug: msg={{ names | map(attribute='ctc') | map('float') }}

  # Appending USD to each CTC value and print
  - debug: msg={{ names | map(attribute='ctc') | product(['USD']) | map('join',' ')}}
```
The Primary use of map filter is with attribute selection from the sequence of elements or objects.

It helps us to select only the values we want from the huge dataset.

In the preceding playbook, you can see we have various examples of map attribute selection.

The last three examples have additional map another filter like upper and float etc.

The join is used to format the output to single-line string which is otherwise a list.  As map returns a list by default.

Here is the output of this playbook

```
PLAY [filter test] ***************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************
ok: [localhost]

TASK [Select and Extract only the cities] ***************************************************************************************************************************************************
ok: [localhost] => {
    "msg": [
        "LasVegas",
        "Boston",
        "Chennai"
    ]
}

TASK [Select only Attributes from List of Objects - Selecting only mobile numbers] ***********************************************************************************************
ok: [localhost] => {
    "msg": [
        "+1-234-31245543",
        "+1-584-31551209",
        "+919876543210"
    ]
}

TASK [debug] *********************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Paul,Rod,Sarav"
}

TASK [debug] *********************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Thompson,Johnson,AK"
}

TASK [debug] *********************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": [
        "THOMPSON",
        "JOHNSON",
        "AK"
    ]
}

TASK [debug] *********************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": [
        100000.0,
        300000.0,
        200000.0
    ]
}

TASK [debug] *********************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": [
        "100000 USD",
        "300000 USD",
        "200000 USD"
    ]
}
```

- Ansible Map Examples – using other Filters

The Second but major role of Ansible map filter is to apply an another filter on a sequence of objects/lists.

Let’s suppose you have a list of names you want to convert all of them to an upper case strings.

For this example, you can consider the same dataset we have used in our first playbook.

In fact, we had one task to convert the last name to upper case like this

`- debug: msg={{ names | map(attribute='last') | map('upper') }}`

which resulted in the upper case converting last names like this.

```
TASK [debug] *********************************************************************************************************************************************************************
ok: [localhost] => {
"msg": [
"THOMPSON",
"JOHNSON",
"AK"
]
}
```

This is just a simple example on Applying other filters using map.

There are more advanced uses of this scenario.

- Using Replace filter with Ansible Map

```
---
- name: Ansible Map with Other Filters
  hosts: localhost
  vars: 
    servers: 
      - "AppServer01"
      - "AppServer02"
      - "WebServer01"
      - "WebServer02"
  tasks: 
    - name: Append the Domain Name to the end of the hosts
      debug: 
        msg={{servers|map('lower')|map('regex_replace','^(.+)$','\\1.gritfy.io')}}
```

In the preceding Playbook, you can see we have list of hostnames assigned to the variable named servers

they are in Camel case which needs to be converted to lower case and a valid domain name gritfy.io has to be added at the end of each host.

Here comes the map to the help

`{{servers|map('lower')|map('regex_replace','^(.+)$','\\1.gritfy.io')}}`
		
we use two map filters here, one is to convert them to lower case and another one is to apply regex_replace filter to replace each hostname with a proper domain suffix.

here is the execution output of this playbook look like

you can see that the hostnames are now in lower case with proper domain extensions.

- Using Extract filter in Ansible Map –  with List/Sequence

Extract filter within Ansible map used to map list of indices to list of values. the list of values can be a simple list or sequence or complex hashtable aka dictionary.

let us first see how an ansible map can be used to iterate a simple list

Here is the example playbook where we have a list of values (names) and we want to take certain names based on their positional index.

```
---
- name: filter test
  hosts: localhost
  tasks: 
    - name: Extract Filter with Ansible Map - Select based on indices
      debug: 
        msg={{ [0,2] | map('extract', ['Sarav','Hanu','Gopi']) | list }}
```

Now we are taking only the 0th and 2nd element from the list.

The result would be Sarav, Gopi  as they are on 0 and 2nd place based on the indices value

- Using Extract filter in Ansible Map –  with Dictionary/Hashtable

As we have already seen how Ansible map filter can be used to select or filter an element using numeric index values.

Let us see an example of how it can be used with dictionary also called as hash table

Unlike the simple list, the dictionary would contain string index as most often called as key

Consider the following dataset of employees at Gritfy

```
{ 
"sarav":{ "mobile":"985643210","email":"sarav@gritfy.com", "city":"Coimbatore" }, 
"hanu":{ "mobile":"7865432109","email":"hanu@gritfy.com","city":"Hyderabad" }, 
"gopi":{ "mobile":"9812990123","email":"gopi@gritfy.com","city":"Chennai" } 
}
```

Now I want to perform a few iterations and select different values from this dataset using extract and map

Here is the playbook

```
---
- name:  Ansible Map Extract filter
  hosts: localhost
  vars: 
        employees:
          { 
            "sarav":{ "mobile":"985643210","email":"sarav@gritfy.com", "city":"Coimbatore" }, 
            "hanu":{ "mobile":"7865432109","email":"hanu@gritfy.com","city":"Hyderabad" }, 
            "gopi":{ "mobile":"9812990123","email":"gopi@gritfy.com","city":"Chennai" } 
          }
  tasks: 
    - name: Task1 - Select only Sarav's records from the Dictionary or Hash table
      debug: 
        msg={{ ['sarav']| map('extract', employees ) }}

    - name: Task2 - Select Hanu's record and look for his email ID using Third argument
      debug: 
        msg=" Hanu`s Email ID is {{ ['hanu']| map('extract', employees, 'email') | last}} "
    
    - name: Task3 - Select Gopi's record and look for his City using Third argument
      debug: 
        msg=" Gopi is residing in {{ ['gopi']| map('extract', employees, 'city') | last }} "
```

we have three tasks here

Task1task is to lookup only Sarav’s records from the employees dataset
Task2 is to lookup Hanu’s records first and then to look up his email ID using the third argument
Task3  is to lookup Gopi’s records first and then to select his city using third argument
		
