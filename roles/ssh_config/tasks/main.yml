- name: Configure SSH daemon options
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?{{ item.option }}\s+.*$'
    line: '{{ item.option }} {{ item.value }}'
    state: present
  loop:
    - { option: 'PasswordAuthentication', value: 'yes' }
    - { option: 'PermitEmptyPasswords', value: 'no' }
    - { option: 'PermitRootLogin', value: 'no' }
  notify: Restart SSHD
  check_mode: no

- name: Validate SSHD configuration
  ansible.builtin.command: /usr/sbin/sshd -t
  register: sshd_config_test
  changed_when: false
  failed_when: sshd_config_test.rc != 0
  check_mode: no
