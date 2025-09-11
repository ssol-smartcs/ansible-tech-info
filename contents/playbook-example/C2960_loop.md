
# 複数のターゲット装置に対して設定を投入する場合

## 概要

#### ・ディクショナリのリスト(cs_parameters)をloopで処理することで、複数のターゲット装置に対して異なるパラメータを設定することができます。

## Playbook例
```yaml
---
- name: configure initial setting to multiple cat2960l via SmartCS
  hosts: smartcs
  gather_facts: no

  vars:
    cs_parameters:
      - { tty_no: '1', hostname: 'switch1', ipaddr: '192.168.0.1' }
      - { tty_no: '2', hostname: 'switch2', ipaddr: '192.168.0.2' }

  tasks:
  - name: configure initial setting to multiple cat2960l via SmartCS
    seiko.smartcs.smartcs_tty_command:
      tty: "{{ item.tty_no }}"
      recvchar:
      - ">"
      - "#"
      sendchar:
      - "__NL__"
      - "enable"
      - "configure terminal"
      - "hostname {{ item.hostname }}"
      - "interface vlan1"
      - "ip address {{ item.ipaddr }} 255.255.255.0"
      - "no shutdown"
      - "end"
      - "write memory"
      - "exit"
      - "exit"
    loop: "{{ cs_parameters }}"
   ```
## 説明
・SmartCSのシリアルポート(tty)ごとに、ホスト名/IPアドレスなどのターゲット装置に応じたパラメータをディクショナリのリスト`cs_parameters`として定義します。
・tasks内でloop処理を使用することで、リスト `cs_parameters` で定義されたttyの順番毎に設定するPlaybook例です。

## 動作環境確認
・SmartCS System Software Version 3.1
・SmartCS modules for Ansible Version 1.7.0
・Ansible 2.16.3
