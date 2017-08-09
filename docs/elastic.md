# How to Install Elasticsearch on Centos 7 Using Ansible

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

Create a file *elastic.yml* and containing the following:
```yaml

---
- hosts: elk
  become: true
  tasks:
  # Configure repo with elasticsearch
  - name: Add Elasticsearch GPG key.
    rpm_key:
      key: http://packages.elastic.co/GPG-KEY-elasticsearch
      state: present

  - copy:
      content: |
                [elasticsearch-5.x]
                name=Elasticsearch repository for 5.x packages
                baseurl=https://artifacts.elastic.co/packages/5.x/yum
                gpgcheck=1
                gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
                enabled=1
                autorefresh=1
                type=rpm-md
      dest: /etc/yum.repos.d/elasticsearch.repo

  # Make sure permissions are correct
  - name: Make executable
    file: 
      path: /etc/yum.repos.d/elasticsearch.repo
      owner: root
      group: root
      mode: 0644
      state: file 
      recurse: no

  # Install
  - name: Install elasticsearch
    yum: name=elasticsearch state=present

  # 1G heap for test only
  # Min Heap size
  - name: change Elastic Search Heap size to 1G
    lineinfile:
      dest: /etc/elasticsearch/jvm.options
      regexp: "^#?-Xms"
      line: "-Xms1g"

  # Max Heap size
  - name: change Elastic Search Heap size to 1G
    lineinfile:
      dest: /etc/elasticsearch/jvm.options
      regexp: "^#?-Xmx"
      line: "-Xmx1g"

  # Make available across the network
  - name: Change allowable hosts
    lineinfile:
      dest: /etc/elasticsearch/elasticsearch.yml
      regexp: "^#?network.host:"
      line: "network.host: \"_local_\""

  # Reload service specs
  - name: Reload specs
    systemd: 
      daemon_reload: yes
      name: elasticsearch.service

  # Start on boot
  - name: Enable on boot
    systemd:
      name: elasticsearch.service
      enabled: yes

  # Start now
  - name: Start now
    systemd:
      name: elasticsearch.service
      state: restarted

  # Open ports
  -  firewalld: 
       port: 9200/tcp
       zone: public
       permanent: true
       state: enabled
       immediate: yes
```

Now execute the ansible script with:

*ansible-playbook -b -K elastic.yml*

You will be promted for the account password. 

This will:

1. Install elasticsearch
1. Set memory limits
1. Install and start a service
1. Set the network to only accept connections from a loopback address
1. Open a port 9200 on the firewall

