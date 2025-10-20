
# 複数のターゲット装置に対して設定を投入する場合

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)

## 概要

* ディクショナリのリスト`cs_parameters`を`loop`で処理することで、複数のターゲット装置に対して異なるパラメータを設定することができます。

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
* SmartCSのシリアルポート(tty)ごとに、ホスト名/IPアドレスなどのターゲット装置に応じたパラメータをディクショナリのリスト`cs_parameters`として定義します。
* tasks内で`loop`処理を使用することで、リスト `cs_parameters` で定義されたttyの順番毎に設定するPlaybook例です。

## 動作環認環境
* SmartCS System Software Version 3.1.1
* SmartCS modules for Ansible Version 1.7.0
* ansible-core 2.16.3

## 変数ファイルを別ディレクトリに置いて設定する場合
* 設定値を別ファイルに記載し、そのファイルをPlaybook内で定義して設定する方法となります。<br>
  設定値を別ディレクトリに置く事で各パラメータに変更が発生した際にPlaybookの修正量が少なくなります。

## 設定ファイル例 (tty1-2_settings.yml)
```yaml
---
cs_parameters:
 
  - { tty_no: '1', hostname: 'switch1', ipaddr: '192.168.0.1' }
  - { tty_no: '2', hostname: 'switch2', ipaddr: '192.168.0.2' }
 ```
## Playbook例
```yaml
---
- name: configure initial setting to multiple cat2960l via SmartCS
  hosts: smartcs
  gather_facts: no
 
  vars_files:
      - config/tty1-2_settings.yml
 
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
* Playbookとは別に設定値を記載した外部ファイルを./config配下に用意します。
* `vars_files:`オプションで外部ファイルの相対パスを指定し、外部ファイル内に記載された設定値をtask内の<br>
`loop`処理を使用することで`cs_parameters`で定義されたttyの順番毎に設定するPlaybook例です


<br><br>

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)
