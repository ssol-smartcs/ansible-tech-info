# 本ページはメンテナンス中です。

[↑目次に戻る](./README.md)
<br>
# ユースケース/サンプルPlaybook

本ページでは、SmartCS x Ansible を活用したユースケース、およびサンプルPlaybook を説明しています。  

`注意事項`  
サンプルPlaybook の内容は、弊社にて確認を行った時点で正常に動作した内容となります。
AnsibleやSmartCS用モジュールのバージョンなど、ご利用の環境によっては記載内容と動作が異なる可能性がございます。  

<br>

## 目次
- [1. 特定のrecvcharを受信した際に特定のsendcharを送信する場合](./README.md#1-特定のrecvcharを受信した際に特定のsendcharを送信する場合)


## 1. 特定のrecvcharを受信した際に特定のsendcharを送信する場合

`smartcs_tty_command` モジュールには`recvchar`と`sendchar`を1対1で対応させるようなオプションがありませんが、`register`変数などと組み合わせることで実現が可能です。
`sendchar`で送信したコマンドの戻り値を`register`変数に格納しておき、期待する文字列が`register`変数内に含まれている場合に続きのコマンドを送信する`task`を別に用意します。

[wait_specific_recvchar.yml](./wait_specific_recvchar.yml) は、SmartCSのtty1に接続されたCisco装置に対して
show running-config GigabitEthernet 0/1 を実行し、GigabitEthernet 0/1が shutdown であれば no shutdown を実行するPlaybook例です。


<br>
<br>

[↑目次に戻る](./README.md)
