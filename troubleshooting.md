[↑目次に戻る](./README.md)
<br>
# トラブルシューティング

本ページでは、SmartCS modules for Ansible を使用して、Playbook 実行時にエラーとなった場合の対応について説明します。
<br>
各項毎にエラー出力と考えられる対処方法について記載しています。  
対処方法については必ずしもエラーを解消できると限りませんが、トラブルシュート時の情報として参考になりましたら幸いです。  
なお、ご利用のAnsibleのバージョンにより、エラーメッセージの内容が異なる可能性がございます。  

>> （各項目の対処方法をより分かりやすく？）


<br>

## 目次
- [1. Unable to connect to port 22 on x.x.x.x](./troubleshooting.md#1-unable-to-connect-to-port-22-on-xxxx)
- [2. timed out](./troubleshooting.md#2-timed-out)
- [3. Error reading SSH protocol banner](./troubleshooting.md#3-error-reading-ssh-protocol-banner)
- [4. The authenticity of host x.x.x.x can't be established.](./troubleshooting.md#4-the-authenticity-of-host-xxxx-cant-be-established)
- [5. Authentication failed.](./troubleshooting.md#5-authentication-failed)
- [6. Bad authentication type](./troubleshooting.md#6-bad-authentication-type)
- [7. Unable to automatically determine host network os.](./troubleshooting.md#7-unable-to-automatically-determine-host-network-os)
- [8. network os cs is not supported](./troubleshooting.md#8-network-os-cs-is-not-supported)
- [9. unable to elevate privilege to enable mode](./troubleshooting.md#9-unable-to-elevate-privilege-to-enable-mode)
- [10. command timeout triggered, timeout value is X secs.](./troubleshooting.md#10-command-timeout-triggered-timeout-value-is-x-secs)
- [11. timeout value X seconds reached while trying to send](./troubleshooting.md#11-timeout-value-x-seconds-reached-while-trying-to-send)
- [12. Ignoring timeout(10) for smartcs_facts](./troubleshooting.md#12-ignoring-timeout10-for-smartcs_facts)


<br>
<br>

## 1. Unable to connect to port 22 on x.x.x.x
#### エラーメッセージ
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "[Errno None] Unable to connect to port 22 on x.x.x.x" 
}
```
#### 対処方法
対象のSmartCS(x.x.x.x)へのSSH接続に失敗しています。  
<br>
SmartCSのSSH サーバが有効化されていない可能性がありますので、以下のコマンドを実行してSmartCSのSSHサーバの状態を確認します。 
```
(0)NS-2250# show service
<sshd>
 status   : disable
 port     : 22
 auth     : basic
 host_key : device_depend
```
`<sshd>` が `status   : disable` となっている場合は、  
以下のコマンドを実行して、SmartCSのSSHサーバを有効化して下さい。
```
(0)NS-2250# enable sshd
```

## 2. timed out
#### エラーメッセージ
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "timed out" 
}
```
#### 対処方法
対象のSmartCSに接続試行中にタイムアウトエラーが発生して接続に失敗しています。  
playbook実行時に使用しているインベントリに登録されているSmartCSのIPアドレスやホスト名、  
および管理ホストとSmartCS間のネットワークをご確認下さい。  
<br>
また、SmartCSのフィルター機能でパケットが廃棄されている可能性があります。  
以下のコマンドを実行してフィルター機能の状態を確認します。
```
(0)NS-2250# show ipfilter input
status : enable

<ipfilter preset input table>
num  target  in    destination        source             prot
  1  ACCEPT  *     0.0.0.0/0          0.0.0.0/0          all  REL,EST
  2  ACCEPT  lo    127.0.0.1          127.0.0.1          all

<ipfilter configurable input table>
num  target  in    destination        source             prot
  1  DROP    *     0.0.0.0/0          0.0.0.0/0          tcp  22
```
`status   : enable` となっていて、SSH接続が破棄されるような設定となっている場合は、  
フィルター機能の無効化、あるいはSSH接続が破棄されないよう必要に応じて設定を追加して下さい。  
<br>
■フィルター機能の無効化
```
(0)NS-2250# disable ipfilter
```
■SSH接続の受信許可(一例)
```
(0)NS-2250# create ipfilter input accept eth1 any any tcp 22
```

## 3. Error reading SSH protocol banner
#### エラーメッセージ
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "Error reading SSH protocol banner[Errno 104] Connection reset by peer" 
}
```
#### 対処方法
対象のSmartCS(x.x.x.x)へのSSH接続に失敗しています。  
<br>
SmartCSのアクセス許可設定でSSHサーバへの接続が許可されていない可能性があります。  
以下のコマンドを実行してSmartCSのアクセス許可設定の状態を確認します。 
```
(0)NS-2250# show allowhost
Service         Address/Mask                            Access tty List
--------------------------------------------------------------------
portd/sshrw     all                                     all
portd/telrw     all                                     all
sshd            all                                     -
telnetd         all                                     -
```
`Service` 列に `sshd` が存在しない、あるいは `sshd` の `Address/Mask` に管理ホストが含まれない場合、  
以下のコマンドを実行して、管理ホストからSmartCSへのSSH接続を許可して下さい。
```
(0)NS-2250# create allowhost <ipaddr/mask> service sshd
```

## 4. The authenticity of host x.x.x.x can't be established.
#### エラーメッセージ
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "paramiko: The authenticity of host 'x.x.x.x ' can't be established. \nThe ecdsa-sha2-nistp521 key fingerprint is b' xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'." 
}
```
#### 対処方法
対象のSmartCS(x.x.x.x)のSSHホスト鍵が管理ホストに登録されておらず、SSH接続に失敗しています。
<br>
対象のSmartCSへ手動でSSH接続を行い、管理ホストにSSHホスト鍵を登録するか、  
`ansible.cfg` の `host_key_checking = False` のコメントアウトを外し、SSHホスト鍵のチェック行わない様にする等の対処をして下さい。
<br>
■手動でSSH接続してホスト鍵を登録する場合
管理ホストから以下のコマンドを実行して、対象のSmartCSへ手動でSSH接続を行います。
```
$ ssh <username>@x.x.x.x
```

手動でSSH接続を行った後に、SSHホスト鍵が登録されていることを下記コマンドで確認します。
```
$ cat ~/.ssh/known_hosts
x.x.x.x ecdsa-sha2-nistp521 XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

■ansible.cfgの設定値を変更する場合
以下のコマンドを実行して、`host_key_checking` の設定状態を確認します。  
`host_key_checking` の行が無い、あるいは `host_key_checking = True` となっている場合は、  
`host_key_checking = False` となるように設定を変更します。
```
$ cat ansible.cfg | grep host_key_checking
#host_key_checking = False
```


## 5. Authentication failed.
#### エラーメッセージ
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "Failed to authenticate: Authentication failed." 
} 
```
#### 対処方法
以下の原因が考えられます。それぞれSmartCSの設定を確認して下さい。

(1)対象のSmartCSに接続許可が無い為、エラーが発生して接続できません。
<br>
SmartCSの接続許可設定を確認し、必要に応じて設定を追加して下さい。
```
(0)NS-2250# show allowhost
(0)NS-2250# create allowhost …
```
(2)Ansibleを実行する管理ホストからSmartCSにログインする際の認証がエラーとなっています。SmartCSにログインするユーザのユーザ名、パスワードを確認して下さい。

## 6. Bad authentication type
#### エラーメッセージ
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "Failed to authenticate: Bad authentication type; allowed types: ['publickey']" 
}
```
#### 対処方法
Ansibleを実行する管理ホストからSmartCSにログインする際のユーザ認証方式に誤りがある為、認証エラーとなっています。SmartCSのSSHサーバユーザ認証方式を管理ホスト側と合わせて下さい。
```
(0)NS-2250# set sshd auth basic
```

## 7. Unable to automatically determine host network os.
#### エラーメッセージ
```
fatal: [x.x.x.x]: FAILED! =>
    "msg": "Unable to automatically determine host network os. Please manually configure ansible_network_os value for this host"
}
```
#### 対処方法
管理ホストからSmartCS に接続する為のネットワーク OS オプションに、"smartcs"が設定されていません。 
Playbook の ansible_network_os に “seiko. smartcs.smartcs"を設定して下さい。

## 8. network os cs is not supported
#### エラーメッセージ
```
fatal: [x.x.x.x]: FAILED! => {
    "msg": "network os cs is not supported"
}
```
#### 対処方法
管理ホストからSmartCS に接続する為のネットワーク OS オプションが設定されていません。 
Playbook の ansible_network_os に “seiko. smartcs.smartcs"を設定して下さい。


## 9. unable to elevate privilege to enable mode
#### エラーメッセージ
```
fatal: [x.x.x.x ]: FAILED! =>
    "msg": "unable to elevate privilege to enable mode, at prompt [b'\\n(2)NS-2250>'] with error: su\r\nPassword:\r\nincorrect password\r\n(2)NS-2250> "
}
```
#### 対処方法
SmartCS にログイン後、装置管理ユーザへの遷移が失敗しました。
Playbookの"ansible_become_password"等で指定したパスワードが正しいかを確認して下さい。


## 10. command timeout triggered, timeout value is X secs.
#### エラーメッセージ
```
"msg": "command timeout triggered, timeout value is 10 secs.\nSee the timeout setting options in the Network Debug and Troubleshooting Guide."
}
```
#### 対処方法
SmartCSへのログイン や、 指定したコマンド を実行する際に タイムアウトが発生し、コマンドの実行がエラーとなりました。
タイムアウト発生時には様々な要因が考えられますので、以下のタイムアウトに関するドキュメントを参照ください。
<br>
- [管理ホスト側のAnsible のタイムアウトに関する各設定 ansible.cfg](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)
- [コネクションプラグインである、network _cliの各設定](https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/network_cli_connection.html)
- [Network Debug and Trouble shooting Guide の Timeout issues](https://docs.ansible.com/ansible/latest/network/user_guide/network_debug_troubleshooting.html#timeout-issues)

## 11. timeout value X seconds reached while trying to send
#### エラーメッセージ
```
"msg": "timeout value 10 seconds reached while trying to send command: b'ttysendwaitset tty 1 timeout 15 nl cr string \\"show version\"'"
}
```
#### 対処方法
Playbookで指定したコマンドの実行において、 タイムアウトが発生し、コマンドの実行がエラーとなりました。
<br>
「command timeout triggered, timeout value is X secs. 」に記載している対処方法や、各モジュールのオプションを確認してください。


## 12. Ignoring timeout(10) for smartcs_facts
#### エラーメッセージ
```
TASK [Gathering Facts] **************************************
[WARNING]: Ignoring timeout(10) for smartcs_facts
ok: [xxx.xxx.xxx.xxx]
```
#### 対処方法
Ansible2.9からネットワークモジュールの facts 収集は、 gather_facts 経由で行われるようになり、 SmartCS の場合は、 smartcs_facts モジュールが内部的に動作します。
<br>
その為、ansible.cfg等で設定されているfacts情報取集のタイムアウト値である、gather_timeout(DEFAULT_GATHER_TIMEOUT)については参照せず、コネクションプラグイン(network_cli)側のタイムアウト値で動作します。
<br>
このワーニングはその内容を警告しており、 Ansible2.9でSmartCSを操作する為の
各モジュール を使った際に、 gather_facts: yes と指定する事で出力されてしまいますが、動作やPlaybook に 問題 ありません。
