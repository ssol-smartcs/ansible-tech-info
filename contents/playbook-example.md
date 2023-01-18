
[↑トップページに戻る](../README.md)
<br>
# ユースケース / サンプルPlaybook

本ページでは、SmartCS x Ansible の連携機能を活用したユースケース、およびサンプルPlaybook を掲載しています。  

`注意事項`  
サンプルPlaybook の内容は、弊社にて確認を行った時点で正常に動作した内容となります。
AnsibleやSmartCS用モジュールのバージョンなど、ご利用の環境によっては記載内容と動作が異なる可能性がございます。  

<br>

## 目次
- [1. 特定のrecvcharを受信した際に特定のsendcharを送信する場合](./playbook-example.md#1-特定のrecvcharを受信した際に特定のsendcharを送信する場合)
- [2. sendcharで指定した文字列の実行結果をファイルとして保存する際に余分な改行を削除する場合](./playbook-example.md#2-sendcharで指定した文字列の実行結果をファイルとして保存する際に余分な改行を削除する場合)

<br>
<br>

## 1. [特定のrecvcharを受信した際に特定のsendcharを送信する場合](./playbook-example/wait_specific_recvchar.md)

* Cisco IOS と連携してコマンド実行結果に特定の文字列が含まれている場合に、特定のコマンドを送信するPlaybookの例となります。

<br>
<br>

## 2. [sendcharで指定した文字列の実行結果をファイルとして保存する際に余分な改行を削除する場合](./playbook-example/convert_nl.md)

* Cisco IOSと連携してコマンド実行結果をファイルに格納し、余分な空白行を削除するPlaybookの例となります。

<br>
<br>

[↑トップページに戻る](../README.md)
