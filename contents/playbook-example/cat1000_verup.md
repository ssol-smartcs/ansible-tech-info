# コンソールからターゲット装置のバージョンアップを実行する場合

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)

## 概要

* `smartcs_tty_command`を使用して、SmartCS 経由でターゲット装置のコンソールからバージョンアップを実行することができます。<br>
* 再起動実行中もコンソールの出力内容を確認できるため、遠隔からでも安全に作業を実施することができます。<br>
[SmartCS × Alaxala × Ansible ハンズオン 演習4.3](https://github.com/ssol-smartcs/ansible-handson/blob/master/SmartCSxALAXALA/4.3-automation_of_firmware_update.md)のページでは、Alaxala 装置でのPlaybook 例もご紹介しています。


## Playbook例

* バージョンアップ用Playbook
```yaml
---
- name: versionup
  hosts: smartcs
  gather_facts: no

  collections:
  - seiko.smartcs

  tasks:
  - name: versionup
    smartcs_tty_command:
      tty: '1'
      nl: cr
      cmd_timeout: 120
      error_detect_on_sendchar: cancel
      error_detect_on_module: failed
      recvchar:
      - 'Username: '
      - 'Password: '
      - 'c1000>'
      - 'c1000#'
      - 'started.'
      sendchar:
      - '__NL__'
      - 'user'
      - 'password'
      - 'enable'
      - 'password'
      - 'terminal width 128'
      - 'archive download-sw /leave-old-sw /reload tftp://user:pass@192.168.0.100/c1000-universalk9-tar.152-7.E7.tar__NOWAIT__:1200'
      - '__NL__'
      - 'user'
      - 'password'
      - 'terminal length 0'
      - 'show version'
      - 'exit'
    register: result

  - name: verup result
    debug: var=result.stdout_lines[11][2]
    failed_when: "'Version 15.2(7)E7' not in result.stdout_lines[11][2]"
```

## 説明

* ターゲット装置からファームウェアファイルを取得し再起動するコマンドを、コンソールから実行します。
* 再起動中に出力される文字列が意図せずrecvcharとマッチすることを防ぐため、`__NOWAIT__` オプションを使用して、再起動が完了するまでrecvcharの受信をチェックしない時間を設定します。
* コマンド実行結果を`register` モジュールでresult という変数に格納し、`failed_when` を使用して期待したバージョン名が`show version` の実行結果に含まれていない場合はエラーと判定します。
* このPlaybook は、SmartCS のtty1 に接続されたCisco 装置に対して、コンソールからバージョンアップを実行するPlaybook 例です。

## 動作確認環境
* SmartCS System Software Version 3.1
* SmartCS modules for Ansible Version 1.4.1
* ansible-core 2.11.2

<br><br>

[↑ ユースケース/サンプルPlaybook に戻る](../playbook-example.md)
