# コンソールに送信する文字列を外部ファイルに記載する場合

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)

## 概要

* `smartcs_tty_command` では、コンソールに送信したい文字列を1行ずつ記述した別ファイルを用意し、`src` オプションによってPlaybook上で呼び出して送信することができます。

## Playbook例

```yaml
---
- name: configure c9200l
  hosts: Cisco
  gather_facts: no

  collections:
  - seiko.smartcs

  tasks:
  - name: login to c9200l
    smartcs_tty_command:
      tty: '1'
      nl: cr
      cmd_timeout: 120
      error_detect_on_sendchar: cancel
      error_detect_on_module: failed
      recvchar:
      - 'Username: '
      - 'Password: '
      - 'c9200l>'
      - 'c9200l#'
      - 'c9200l(config)#'
        sendchar:
      - __NL__
      - username
      - password
      - terminal length 0
      - terminal width 512
      - configure terminal
      - no logging console

  - name: configure c9200l
    smartcs_tty_command:
      tty: '1'
      nl: cr
      cmd_timeout: 120
      error_detect_on_sendchar: cancel
      error_detect_on_module: failed
      error_recvchar_regex:
      - "Unrecognized"
      recvchar:
      - 'c9200l(config)#'
      - 'c9200l(config-vlan)#'
      - 'c9200l(config-if)#'
      src: "commands.txt"

  - name: show config & logout
    smartcs_tty_command:
      tty: '1'
      nl: cr
      cmd_timeout: 120
      error_detect_on_sendchar: cancel
      error_detect_on_module: failed
      custom_response: on
      custom_response_delete_nl: yes
      custom_response_delete_lastline: yes
      recvchar:
      - 'c9200l>'
      - 'c9200l#'
      - 'c9200l(config)#'
      - 'c9200l(config-if)#'
      - 'get started.'
      sendchar:
      - exit
      - exit
      - show running-config__WAIT__:90
      - terminal width 80
      - write memory
      - exit
    register: config

  - name: save config to localhost
    copy:
      content: "{{ config.stdout_lines_custom[2].response | join('\n') }}"
      dest: ./c9200l/c9200l_config.txt
```

## srcファイル例（commands.txt）
```
!
vlan 10
 name network1
vlan 20
 name network2
!
!
interface GigabitEthernet1/0/1
 description Eth1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 switchport trunk native vlan 10
 storm-control broadcast level 10.00
 storm-control multicast level 10.00
 storm-control unknown-unicast level 10.00
 storm-control action shutdown
 storm-control action trap
!
interface GigabitEthernet1/0/2
 description Eth2
 switchport mode access
 switchport access vlan 20
 storm-control broadcast level 10.00
 storm-control multicast level 10.00
 storm-control unknown-unicast level 10.00
 storm-control action shutdown
 storm-control action trap
!
（略）
```

## 説明

* Playbook とは別に、コンソールに送信したい文字列を1行ずつ記述した外部ファイルを用意します。
* `src` オプションで、外部ファイルの絶対パス、またはPlaybook の保存先ディレクトリからの相対パスを指定することで、外部ファイルに記載された文字列をコンソールへ送信します。
* 文字列の送信後、`show running-config` 等の実行結果を`register` モジュールでconfig という変数に格納し、`copy` モジュールを使用して`show running-config` の返り値のみをファイルとして保存します。
* このPlaybook は、SmartCS のtty1 に接続されたCisco 装置のコンソールに対して、外部ファイル`commands.txt` に記載された文字列を送信後、設定内容をローカルホストに`c9200l_config.txt` ファイルとして保存するPlaybook 例です。


## 動作確認環境
* SmartCS System Software Version 3.0
* SmartCS modules for Ansible Version 1.4.1
* Ansible 2.10.6

<br><br>

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)
