[↑目次に戻る](./README.md)
<br>
# トラブルシューティング

本ページでは、SmartCS modules for Ansible を使用して、Playbook 実行時にエラーとなった場合の対応について説明します。
<br>
各項毎にエラー出力と考えられる対処方法について記載しています。対処方法については必ずしもエラーを解消できると限りませんが、トラブルシュート時の情報として参考になり
ましたら幸いです。

（使用するバージョンによってエラー表記が違うかも的な分章も入れる）
（各項目の対処方法をより分かりやすく？）
<br>

## 目次

### 1. Unable to connect to port 22 on x.x.x.x
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "[Errno None] Unable to connect to port 22 on x.x.x.x" 
}
```
対象のSmartCSに接続できません。管理ホストに登録しているSmartCSのIPアドレス、ホスト名や、管理ホストとSmartCSの間のネットワークをご確認下さい。
<br>
また、SmartCSのSSH サーバが有効化されていない可能性があります。以下のコマンドを実行して、SmartCSのSSHサーバを有効化して下さい。
```
(0)NS-2250# enable sshd
```

### 2. timed out
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "timed out" 
}
```
対象のSmartCSに接続試行中にタイムアウトエラーが発生して接続できません。管理ホストとSmartCS間のネットワークをご確認下さい。
<br>
また、SmartCSのフィルター機能でパケットが廃棄されている可能性があります。
<br>
以下のコマンドを実行して、管理ホストからのパケットが正しくSmartCSに届く設定となっているかを確認し、必要に応じて設定を追加して下さい。
```
(0)NS-2250# show ipfilter input
(0)NS-2250# create ipfilter input accept …
```

### 3. Error reading SSH protocol banner
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "Error reading SSH protocol banner[Errno 104] Connection reset by peer" 
}
```
対象のSmartCSのttyについて、接続可能なRWセッション数を超過している可能性があります。portd設定を確認し、必要に応じて設定を変更して下さい。
```
(0)NS-2250# show portd tty
(0)NS-2250# set portd tty x limit rw 2 ro 3
```

### 4. The authenticity of host x.x.x.x can't be established.
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "paramiko: The authenticity of host 'x.x.x.x ' can't be established. \nThe ecdsa-sha2-nistp521 key fingerprint is b' xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'." 
}
```
SmartCSのSSHホスト鍵が管理ホストPCに登録されていません。
<br>
SSH接続を行い、ホスト鍵を登録するか、ansible.cfgの”host_key_checking = False”のコメントアウトを外し、SSHホスト鍵のチェック行わない様にする等、確認をして下さい。
<br>

### 5. Authentication failed.
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "Failed to authenticate: Authentication failed." 
} 
```
以下の原因が考えられます。それぞれSmartCSの設定を確認して下さい。

(1)対象のSmartCSに接続許可が無い為、エラーが発生して接続できません。
<br>
SmartCSの接続許可設定を確認し、必要に応じて設定を追加して下さい。
```
(0)NS-2250# show allowhost
(0)NS-2250# create allowhost …
```
(2)Ansibleを実行する管理ホストからSmartCSにログインする際の認証がエラーとなっています。SmartCSにログインするユーザのユーザ名、パスワードを確認して下さい。

### 6. Bad authentication type
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "Failed to authenticate: Bad authentication type; allowed types: ['publickey']" 
}
```
Ansibleを実行する管理ホストからSmartCSにログインする際のユーザ認証方式に誤りがある為、認証エラーとなっています。SmartCSのSSHサーバユーザ認証方式を管理ホスト側と合わせて下さい。
```
(0)NS-2250# set sshd auth basic
```

### 7. Unable to automatically determine host network os.

```
fatal: [x.x.x.x]: FAILED! =>
    "msg": "Unable to automatically determine host network os. Please manually configure ansible_network_os value for this host"
}
```

管理ホストからSmartCS に接続する為のネットワーク OS オプションに、"smartcs"が設定されていません。 
Playbook の ansible_network_os に “seiko. smartcs.smartcs"を設定して下さい。

### 8. network os cs is not supported
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "network os cs is not supported"
}
```

管理ホストからSmartCS に接続する為のネットワーク OS オプションが設定されていません。 
Playbook の ansible_network_os に “seiko. smartcs.smartcs"を設定して下さい。


### 9. unable to elevate privilege to enable mode
```
fatal: [x.x.x.x ]: FAILED! =>
    "msg": "unable to elevate privilege to enable mode, at prompt [b'\\n(2)NS-2250>'] with error: su\r\nPassword:\r\nincorrect password\r\n(2)NS-2250> "
}
```
SmartCS にログイン後、装置管理ユーザへの遷移が失敗しました。
Playbookの"ansible_become_password"等で指定したパスワードが正しいかを確認して下さい。


### 10.command timeout triggered, timeout value is X secs.
```
"msg": "command timeout triggered, timeout value is 10 secs.\nSee the timeout setting options in the Network Debug and Troubleshooting Guide."
}
```
SmartCSへのログイン や、 指定したコマンド を実行する際に タイムアウトが発生し、コマンドの実行がエラーとなりました。
タイムアウト発生時には様々な要因が考えられますので、以下のタイムアウトに関するドキュメントを参照ください。
<br>
- [管理ホスト側のAnsible のタイムアウトに関する各設定 ansible.cfg](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)
- [コネクションプラグインである、network _cliの各設定](https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/network_cli_connection.html)
- [Network Debug and Trouble shooting Guide の Timeout issues](https://docs.ansible.com/ansible/latest/network/user_guide/network_debug_troubleshooting.html#timeout-issues)

### 11. timeout value X seconds reached while trying to send
```
"msg": "timeout value 10 seconds reached while trying to send command: b'ttysendwaitset tty 1 timeout 15 nl cr string \\"show version\"'"
}
```
Playbookで指定したコマンドの実行において、 タイムアウトが発生し、コマンドの実行がエラーとなりました。
<br>
「command timeout triggered, timeout value is X secs. 」に記載している対処方法や、各モジュールのオプションを確認してください。


### 12. Ignoring timeout(10) for smartcs_facts
```
TASK [Gathering Facts] **************************************
[WARNING]: Ignoring timeout(10) for smartcs_facts
ok: [xxx.xxx.xxx.xxx]
```
Ansible2.9からネットワークモジュールの facts 収集は、 gather_facts 経由で行われるようになり、 SmartCS の場合は、 smartcs_facts モジュールが内部的に動作します。
<br>
その為、ansible.cfg等で設定されているfacts情報取集のタイムアウト値である、gather_timeout(DEFAULT_GATHER_TIMEOUT)については参照せず、コネクションプラグイン(network_cli)側のタイムアウト値で動作します。
<br>
このワーニングはその内容を警告しており、 Ansible2.9でSmartCSを操作する為の
各モジュール を使った際に、 gather_facts: yes と指定する事で出力されてしまいますが、動作やPlaybook に 問題 ありません。