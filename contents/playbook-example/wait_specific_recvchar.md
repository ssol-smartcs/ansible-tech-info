# 特定のrecvcharを受信した際に特定のsendcharを送信する場合

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)

## 概要

* `smartcs_tty_command` モジュールには`recvchar`と`sendchar`を1対1で対応させるようなオプションがありませんが、`register`変数などと組み合わせることで実現が可能です。  


## Playbook例

```yaml
---
- name: check status and enable interface
  hosts: smartcs
  gather_facts: no

  tasks:
  - name: show running config
    seiko.smartcs.smartcs_tty_command:
      tty: '1'
      error_detect_on_module: failed
      recvchar:
      - 'Password: '
      - 'Switch>'
      - 'Switch#'
      sendchar:
      - '__NL__'
      - 'secret'
      - 'enable'
      - 'secret'
      - 'show running-config interface GigabitEthernet 0/1'
    register: result

  - name: enable interface
    seiko.smartcs.smartcs_tty_command:
      tty: '1'
      error_detect_on_module: failed
      recvchar:
      - 'started.'
      - 'Switch#'
      - 'Switch(config)#'
      - 'Switch(config-if)#'
      sendchar:
      - '__NL__'
      - 'configure terminal'
      - 'interface GigabitEthernet 0/1'
      - 'no shutdown'
      - 'end'
      - 'exit'
    when: "' shutdown' in result.stdout_lines[4]" 

  vars:
  - ansible_connection: ansible.netcommon.network_cli
  - ansible_network_os: seiko.smartcs.smartcs
  - ansible_command_timeout: 60
  - ansible_user: ansible
  - ansible_password: ansible
```

## 説明

*  `sendchar` で送信したコマンドの戻り値を`register` 変数に格納しておき、期待する文字列が`register` 変数内に含まれている場合に続きのコマンドを送信する`task` を別に用意します。

* このPlaybook は、SmartCSのtty1に接続されたCisco装置に対して`show running-config GigabitEthernet 0/1` を実行し、`GigabitEthernet 0/1` が`shutdown` であれば`no shutdown` を実行するPlaybook 例です。


## 動作確認環境
* SmartCS System Software Version 2.2
* SmartCS modules for Ansible Version 1.4.1
* Ansible 2.10.15

<br><br>

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)
