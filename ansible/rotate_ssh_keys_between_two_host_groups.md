# Rotate SSH Keys between two host groups

```yaml
# Rotate SSH Keys between two host groups
#
# Inspired by http://vicendominguez.blogspot.kr/2014/05/ansible-generating-pub-key-file-and.html
#

- name: Remove current .ssh/
  hosts: host_group1, host_group2
  remote_user : "{{ user }}"
  tasks:
    - file: path={{item}} state=absent
      with_items:
        - ~/.ssh
  tags: ssh

- name: Generate SSH keys and fetch public key to local /tmp
  hosts: host_group1, host_group2
  remote_user : "{{ user }}"
  tasks:
    - file: path={{item}} state=absent
      with_items:
        - ~/.ssh/id_rsa
        - ~/.ssh/id_rsa.pub
    - command: ssh-keygen -N '' -f ~/.ssh/id_rsa
    - fetch: src=~/.ssh/id_rsa.pub dest="/tmp/{{inventory_hostname}}.pub" flat=yes
  tags: ssh

- name: Add `host_group1` authorized Keys to `host_group2`
  hosts: host_group2
  remote_user : "{{ user }}"
  tasks:
    - authorized_key: user="{{ user }}" key="{{ lookup('file', '/tmp/{{ item }}.pub') }}" state=present exclusive=no
      with_items: "{{groups['host_group1']}}"

    - known_hosts: name="{{item}}" state=present key="{{ lookup('pipe', 'ssh-keyscan -t rsa {{ item }}') }}"
      with_items: "{{groups['host_group1']}}"

    - local_action: "command rm -f /tmp/{{ item }}.pub"
      with_items: "{{groups['host_group1']}}"
  tags: ssh

- name: Add `host_group2` authorized Keys to `host_group1`
  hosts: host_group1
  remote_user : "{{ user }}"
  tasks:
    - authorized_key: user="{{ user }}" key="{{ lookup('file', '/tmp/{{ item }}.pub') }}" state=present exclusive=no
      with_items: "{{groups['host_group2']}}"

    - known_hosts: name="{{item}}" state=present key="{{ lookup('pipe', 'ssh-keyscan -t rsa {{ item }}') }}"
      with_items: "{{groups['host_group2']}}"

    - local_action: "command rm -f /tmp/{{ item }}.pub"
      with_items: "{{groups['host_group2']}}"
  tags: ssh
```

https://gist.github.com/kyungw00k/d56547b1e30b5d72316e7df0e9d99f92