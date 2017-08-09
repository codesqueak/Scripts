# How to Install Oracle Java on Centos 7 Using Ansible

## Initial Setup

1. Make sure you have access to the target machine using ssh
1. Install ansible locally (*sudo yum install ansible*)
1. Add a host entry to */etc/ansible/hosts* for the target machine(s)

For example, to add a group of machines to the ELK group, add an entry to the *hosts* file such as:
```
[elk]
elk1 ansible_ssh_host=172.16.1.7
```

### Install

Create a file *java.yml* and containing the following:
```yaml

---
- hosts: elk
  become: true
  vars:
    # This is only for release 144. Need to change per release
    jdk_archive: jdk-8u144-linux-x64.tar.gz
    jdk_local_dir: jdk1.8.0_144
    jdk_version: 8u144-b01
    jdk_path_key: 090f390dda5b47b9b721c7dfaa008135
    # Constant from here on ...
    download_folder: /opt/oracle
    bin_dir: /usr/bin
    java_name: "{{ download_folder }}/{{ jdk_local_dir }}"
    java_archive: "{{ download_folder }}/{{ jdk_archive }}"
    jdk_tarball_url: "http://download.oracle.com/otn-pub/java/jdk/{{ jdk_version }}/{{ jdk_path_key }}/{{ jdk_archive }}"

  tasks:
  - name: Create Directory structure
    command: mkdir -p {{download_folder}}
  - name: Create dir part II
    command: mkdir -p {{java_name}}

  - name: Download Java
    get_url: url={{ jdk_tarball_url }}  dest={{ java_archive }} headers="Cookie:' gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie'" validate_certs=no owner=root group=root mode=744

  - name: Unpack archive
    action: shell tar -xzvf {{java_archive}} -C {{download_folder}}

  - name: Fix ownership
    file: state=directory path={{java_name}} owner=root group=root recurse=yes

  - name: Make Java available for system by updating alternatives
    command: "{{ item }}"
    with_items:
    - alternatives --install /usr/bin/java java {{ java_name }}/bin/java 9999
    - alternatives --install /usr/bin/javac javac {{ java_name }}/bin/javac 9999
    - alternatives --set java {{ java_name }}/bin/java
    - alternatives --set javac {{ java_name }}/bin/javac
    - alternatives --install /usr/bin/jar jar {{ java_name }}/bin/jar 9999
    # Hack. No idea why we have to do this or the --install doesn't work
    - rm -f /usr/bin/jar
    - rm -f /var/lib/alternatives/jar
    - alternatives --install /usr/bin/jar jar {{ java_name }}/bin/jar 9999
    - alternatives --set jar {{ java_name }}/bin/jar

  # Copy /etc/profile.d/oracle_jdk.sh with content
  - copy:
      content: |
                #!/bin/bash
                export JDK_HOME={{ java_name }}
                export JAVA_HOME={{ java_name }}
                export JRE_HOME={{ java_name }}/jre
                export PATH=$PATH:{{ java_name }}/bin:{{ java_name }}/jre/bin
      dest: /etc/profile.d/oracle_jdk.sh

  # Make sure permissions are correct
  - name: Make executable
    file: path=/etc/profile.d/oracle_jdk.sh owner=root group=root mode=0644 state=file recurse=no

```

Now execute the ansible script with:

*ansible-playbook -b -K java.yml*

You will be promted for the account password. 

This will:

1. Install Java JDK 8.0_144 
1. Make this Java version as the first alternative to use if other JDK's are installed
1. Make JDK environment variables available system wide

Note: If you want to use a different JDK version, the following declarartions will need modifying:

```yaml
    jdk_archive: jdk-8u144-linux-x64.tar.gz
    jdk_local_dir: jdk1.8.0_144
    jdk_version: 8u144-b01
    jdk_path_key: 090f390dda5b47b9b721c7dfaa008135
```

