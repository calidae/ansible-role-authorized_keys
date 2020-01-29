Authorized Keys
=========

Manage ssh authorized keys through the `authorized_key` Ansible module.

Role Variables
--------------
Default variables:
```
authorized_keys_present: {root: []}
authorized_keys_banned: []
authorized_keys_exclusive: {}
```

`authorized_keys_present` can be used to state the authorized keys for every user in every host. By default it includes the root user with no keys.

`authorized_keys_banned` can be used to remove a set of authorized keys for any user in `authorized_keys_present`. It can be used to remove keys accross all the inventory once thay have been rotated or compromised or if their issuers have been revoked all access. By default it will remove all keys listed here for the root user.

If in need of a granular and precise control, use `authorized_keys_exclusive`. It will both install the stated keys and also remove any additional key that is found in the server, providing consistent access control despite the configurational drift. If the key list is empty this option will **not** remove all keys for a given user to avoid loosing all ssh access.

Example Playbook
----------------

Naive example:
```
# example.yml

- hosts: web_servers
  roles:
    - calidae.authorized_keys
  vars:
    authorized_keys_present:
      root:
        - '{{ lookup("file", "public_keys/alice") }}'
      ubuntu:
        - '{{ lookup("file", "public_keys/alice") }}'
        - '{{ lookup("file", "public_keys/beth") }}'
    authorized_keys_banned:
      - '{{ lookup("file", "public_keys/alice_old") }}'
      - '{{ lookup("file", "public_keys/claire") }}'
      - 'ssh-rsa AAAAB3Nza..(some bytes eluded)..bCRkh7ReBbpx daisy@office'

- hosts: db_servers
  roles:
    - calidae.authorized_keys
  vars:
    authorized_keys_exclusive:
      root:
        - '{{ lookup("file", "public_keys/alice") }}'
        - '{{ lookup("file", "public_keys/emilie") }}'
      ubuntu: [] # this will not add nor remove any key for that user!
```

`ansible-playbook example.yml --diff --check`

Advanced Example:

Put your public keys in a files directory of a custom role depending on calidae.authorized_keys. State `authorized_keys_banned` as a role variable (not a default) so it is hard to be overriden. Use either `authorized_keys_present` or `authorized_keys_exclusive` to add keys depening on the control you need for every host.

```
# roles/myorg.authorized_keys/meta/main.yml
dependencies:
  - calidae.authorized_keys
```
```
# roles/myorg.authorized_keys/vars/main.yml
alice: '{{ lookup("file", "public_keys/alice") }}'
beth: '{{ lookup("file", "public_keys/beth") }}'
claire: '{{ lookup("file", "public_keys/claire") }}'
daisy: 'ssh-rsa AAAAB3Nza..(some bytes eluded)..bCRkh7ReBbpx daisy@office'
emilie: '{{ lookup("file", "public_keys/emilie") }}'
authorized_keys_banned:
  - '{{ daisy }}'
```
```
# group_vars/web_servers.yml
authorized_keys_present:
  root:
    - '{{ alice }}'
  ubuntu:
    - '{{ beth }}'
    - '{{ claire }}'
```
```
# group_vars/sftp.yml
authorized_keys_present:
  root:
    - '{{ alice }}'
    - '{{ emilie }}'
  customer:
    - 'ssh-rsa AAAAB3Nza..(some bytes eluded)..t1F0Q5Y2AN customer@somesystem'
```
```
# group_vars/db_servers.yml
authorized_keys_exclusive:
  root:
    - '{{ alice }}'
```
```
# plays/site.yml
- hosts: all
  roles:
    - myorg.authorized_keys
```
`ansible-playbook plays/site.yml`

License
-------

BSD

Author Information
------------------

Calidae https://www.calidae.com
