
<br>
# トラブルシューティング
本ページでは、SmartCS modules for Ansible を使用して、Playbook 実行時にエラーとなった場合の対応について説明します。
各項毎にエラー出力と考えられる対処方法について記載しています。対処方法については必ずしもエラーを解消できると限りませんが、トラブルシュート時の情報として参考になり
ましたら幸いです。
（使用するバージョンによってエラー表記が違うかも的な分章も入れる）
<br>

## 目次

### 7. Unable to automatically determine host network os.
fatal: [x.x.x.x]: FAILED! =>
"msg": "Unable to automatically determine host network os. Please manually
configure ansible_network_os value for this host"
}
管理ホストからSmartCS に接続する為のネットワーク OS オプションに、"smartcs"が設定されていません。 
Playbook の ansible_network_os に “seiko. smartcs.smartcs"を設定して下さい。

### 8. network os cs is not supported
fatal: [x.x.x.x]: FAILED! => {
    "msg": "network os cs is not supported"
}
管理ホストからSmartCS に接続する為のネットワーク OS オプションが設定されていません。 
Playbook の ansible_network_os に “seiko. smartcs.smartcs"を設定して下さい。


### 9. unable to elevate privilege to enable mode
fatal: [x.x.x.x ]: FAILED! =>
"msg": "unable to elevate privilege to enable mode, at prompt [b'\\n(2)NS-2250>
'] with error: su\r\nPassword:\r\nincorrect password\r\n(2)NS-2250> "
}
SmartCS にログイン後、装置管理ユーザへの遷移が失敗しました。
Playbookの"ansible_become_password"等で指定したパスワードが正しいかを確認して下さい。


### 10.command timeout triggered, timeout value is X secs.
"msg": "command timeout triggered, timeout value is 10 secs.\nSee the timeout
setting options in the Network Debug and Troubleshooting Guide."
}
SmartCSへのログイン や、 指定したコマンド を実行する際に タイムアウトが発生し、コマンドの実行がエラーとなりました。
タイムアウト発生時には様々な要因が考えられますので、
以下のタイムアウトに関するドキュメントを参照ください。
・管理ホスト側のAnsible のタイムアウトに関する各設定 ansible.cfg
https://docs.ansible.com/ansible/latest/reference_appendices/config.html
・コネクションプラグインである、network _cliの各設定
https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/network_cli_connection.html
・Network Debug and Trouble shooting Guide の Timeout issues
https://docs.ansible.com/ansible/latest/network/user_guide/network_debug_troubleshooting.html#timeout-issues

### 11. timeout value X seconds reached while trying to send
}
"msg": "timeout value 10 seconds reached while trying to send command:
b'ttysendwaitset tty 1 timeout 15 nl cr string \\"show version\"'"
}
Playbookで指定したコマンドの実行において、 タイムアウトが発生し、コマンドの実行がエラーとなりました。
「command timeout triggered, timeout value is X secs. 」に記載している対処方法や、
各モジュールのオプションを確認してください。


### 12. Ignoring timeout(10) for smartcs_facts
TASK [Gathering Facts] **************************************
[WARNING]: Ignoring timeout(10) for smartcs_facts
ok: [xxx.xxx.xxx.xxx]
Ansible2.9からネットワークモジュールの facts 収集は、 gather_facts 経由で行われるようになり、 
SmartCS の場合は、 smartcs_facts モジュールが内部的に動作します。
その為、ansible.cfg等で設定されているfacts情報取集のタイムアウト値である、
gather_timeout(DEFAULT_GATHER_TIMEOUT)については参照せず、コネクションプラグイン(network_cli)側のタイムアウト値で動作します。
このワーニングはその内容を警告しており、 Ansible2.9でSmartCSを操作する為の
各モジュール を使った際に、 gather_facts: yes と指定する事で出力されてしまいますが、動作やPlaybook に 問題 ありません。
