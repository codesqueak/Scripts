# How to Install Kibana on Centos 7 Using Ansible

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
                [kibana-5.x]
                name=Kibana repository for 5.x packages
                baseurl=https://artifacts.elastic.co/packages/5.x/yum
                gpgcheck=1
                gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
                enabled=1
                autorefresh=1
                type=rpm-md
      dest: /etc/yum.repos.d/kibana.repo

  # Make sure permissions are correct
  - name: Make executable
    file: 
      path: /etc/yum.repos.d/kibana.repo
      owner: root
      group: root
      mode: 0644
      state: file 
      recurse: no

  # Install
  - name: Install kibana
    yum: name=kibana state=present

  # Server name
  - name: Set Server Name
    lineinfile:
      dest: /etc/kibana/kibana.yml
      regexp: "^#?server.name:"
      line: "server.name: \"Kibana-1\""

  # Point to elasticsearch
  - name: Elasticsearch
    lineinfile:
      dest: /etc/kibana/kibana.yml
      regexp: "^#?elasticsearch.url:"
      line: "elasticsearch.url: \"http://localhost:9200\""

  # Reload service specs
  - name: Reload specs
    systemd: 
      daemon_reload: yes
      name: kibana.service

  # Start on boot
  - name: Enable on boot
    systemd:
      name: kibana.service
      enabled: yes

  # Start now
  - name: Start now
    systemd:
      name: kibana.service
      state: restarted

  # Open ports
  -  firewalld: 
       port: 5601/tcp
       zone: public
       permanent: true
       state: enabled
       immediate: yes

```

Now execute the ansible script with:

*ansible-playbook -b -K kibana.yml*

You will be promted for the account password. 

This will:

1. Install Kibana
1. Install and start a service
1. Open a port 5601 on the firewall


