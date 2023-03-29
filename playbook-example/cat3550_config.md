# ベンダーモジュールと連携してコンソールから設定を投入する場合

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)

## 概要

* 装置ベンダー製のAnsible モジュール（以下、ベンダーモジュール）が用意されているターゲット装置については、ベンダーモジュールと`smartcs_tty_command` を連携して、SmartCS 経由でコンソールからもPlaybook を実行することができます。<br>
詳細については[演習4.1 STEP4](ansible-handson/SmartCSxIOS/4.1-automation_of_operation_error_recovery.md)のページもご参照ください。


## Playbook例

* コンソールログイン用Playbook（console_login.yml）
```yaml
---
- name: Login from Console using SmartCS
  hosts: smartcs
  gather_facts: no

  tasks:
  - name: login cat3550
    seiko.smartcs.smartcs_tty_command:
      tty: '1'
      error_detect_on_module: failed
      recvchar:
      - 'Username: '
      - 'Password: '
      - 'Cat3550>'
      sendchar:
      - '__NL__'
      - 'cisco'
      - 'password'
```

* オペレーション用Playbook（console_add_acl.yml）
```yaml
---
- name: add acl configration
  hosts: ios_sshxpt
  gather_facts: no

  tasks:
  - name: configure access list
    cisco.ios.ios_acls:
      config:
      - afi: ipv4
        acls:
        - name: add access-list
          acl_type: extended
          aces:
          - sequence: 10
            grant: permit
            protocol_options:
              ip: yes
            source:
              host: "{{ src_ipaddr }}"
            destination:
              host: "{{ dest_ipaddr }}"
          - sequence: 20
            grant: deny
            protocol_options:
              ip: yes
            source:
              any: yes
            destination:
              any: yes
      state: merged

  - name: set access-list to interface
    cisco.ios.ios_acl_interfaces:
      config:
      - name: Vlan1
        access_groups:
        - afi: ipv4
          acls:
          - name: access_ansible_host_only
            direction: in
      state: merged

  - name: always save running to startup
    cisco.ios.ios_config:
      save_when: always
```

* コンソールログアウト用Playbook（console_logout.yml）
```yaml
---
- name: Logout from Console using SmartCS
  hosts: smartcs
  gather_facts: no

  tasks:
  - name: logout cat3550
    seiko.smartcs.smartcs_tty_command:
      tty: '1'
      error_detect_on_module: failed
      recvchar:
      - 'Cat3550#'
      - 'Press RETURN to get started.'
      sendchar:
      - 'exit'
```

* 制御用Playbook
```yaml
---
- name: login by console
  import_playbook: console_login.yml

- name: add acl configration
  import_playbook: console_add_acl.yml

- name: logout by console
  import_playbook: console_logout.yml
```

## Inventoryファイル例

* /etc/ansible/hosts
```
[seiko]
smartcs ansible_host=192.168.129.1 ansible_user=user01 ansible_password=secret01

[cisco]
ios ansible_host=192.168.128.1 ansible_user=cisco ansible_password=password smartcs_tty=1
ios_sshxpt ansible_host=192.168.129.1 ansible_user=port01 ansible_password=secret01 ansible_port=9301

[seiko:vars]
ansible_connection=ansible.netcommon.network_cli
ansible_network_os=seiko.smartcs.smartcs

[cisco:vars]
ansible_become=yes
ansible_become_method=enable
ansible_become_password=secret3550
ansible_connection=ansible.netcommon.network_cli
ansible_network_os=cisco.ios.ios
```


## 説明

* コンソールへのログインとログアウトには`smartcs_tty_command` モジュールを使用します。
** `hosts` にはsmartcs(SmartCS) を指定します。
** `Inventory` ファイルの`ansible_user` 、`ansible_password` には、SmartCS のポートユーザを指定します。
* オペレーションにはベンダーモジュールを使用します。
** `hosts` にはios_sshxpt(SmartCS) を指定します。
** `Inventory` ファイルの`ansile_user` 、`ansible_password` には、SmartCS のポートユーザを指定します。
** `Inventory` ファイルの`ansible_ssh_port` には、SmartCS のトランスペアレント接続機能で使用するポートを指定します。
* 制御用Playbook を実行すると、ログイン、オペレーション、ログアウトの3つのPlaybook が連続して実行されます
* このPlaybook は、SmartCS のtty1 に接続されたCisco 装置に対して、コンソールからACL の設定を追加するPlaybook 例です。


## 動作確認環境
* SmartCS System Software Version 2.2
* SmartCS modules for Ansible Version 1.2.1
* Ansible 2.10.6

<br><br>

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)
