# sendcharで指定したコマンドの実行結果から余分な改行を削除する場合

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)

## 概要

* Ansibleから`show`コマンドなどを実行したとき、戻り値に余分な空白行が含まれる場合があります。<br>
`copy`モジュールを使用して、戻り値をファイルとして保存した後、`replace`モジュールでファイル内の空白行を削除（置換）することで、手動での実行結果と同じ内容を得ることができます。


## Playbook例

```yaml
---
- hosts: smartcs
  gather_facts: no

  tasks:
  - name: execute show version on IOS
    seiko.smartcs.smartcs_tty_command:
      tty: '1'
      cmd_timeout: 60
      recvchar: 
      - 'Switch>'
      - 'Switch#'
      - 'started.'
      sendchar:
      - __NL__
      - 'enable'
      - 'terminal length 0'
      - 'show version'
      - 'exit'
    register: result

  - name: create textfile
    copy:
      content: '{{ result.stdout[3] }}'
      dest: ./log/showversion.txt

  - name: convert LFLF to LF
    replace:
      path: ./log/showversion.txt
      regexp: '\n\n'
      replace: '\n'

  vars:
  - ansible_connection: ansible.netcommon.network_cli
  - ansible_network_os: seiko.smartcs.smartcs
  - ansible_command_timeout: 60
  - ansible_user: ansible
  - ansible_password: ansible
```

## 説明

* `sendchar`で送信したコマンドの戻り値を`register`変数に格納しておき、`copy`モジュールでテキストファイルとして保存します。
* `replace`モジュールを使用して、保存したテキストファイル内の`\n\n`を`\n`に置換することで、余分な空白行を削除します。
* このPlaybook は、SmartCSのtty1に接続されたCisco装置に対して`show version`を実行し、実行結果を./logディレクトリに`showversion.txt`ファイルとして保存後、内容の余分な改行を置換するPlaybook 例です。


## 動作確認環境
* SmartCS System Software Version 3.1
* SmartCS modules for Ansible Version 1.4.1
* Ansible 2.10.15

<br><br>

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)
