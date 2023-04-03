
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
- [2. sendcharで実行したコマンドの実行結果から余分な改行を削除する場合](./playbook-example.md#2-sendcharで指定したコマンドの実行結果から余分な改行を削除する場合)
- [3. ベンダーモジュールと連携してコンソールから設定を投入する場合](./playbook-example.md#3-ベンダーモジュールと連携してコンソールから設定を投入する場合)
- [4. ベンダーモジュールと連携してコンソールから情報を取得する場合](./playbook-example.md#4-ベンダーモジュールと連携してコンソールから情報を取得する場合)
- [5. コンソールからターゲット装置のバージョンアップを実行する場合](./playbook-example.md#5-コンソールからターゲット装置のバージョンアップを実行する場合)

<br>
<br>

## 1. [特定のrecvcharを受信した際に特定のsendcharを送信する場合](./playbook-example/wait_specific_recvchar.md)

* Cisco IOS と連携してコマンド実行結果に特定の文字列が含まれている場合に、特定のコマンドを送信するPlaybookの例となります。

<br>
<br>

## 2. [sendcharで指定したコマンドの実行結果から余分な改行を削除する場合](./playbook-example/convert_nl.md)

* Cisco IOSと連携してコマンド実行結果をファイルに格納し、余分な空白行を削除するPlaybookの例となります。

<br>
<br>

## 3. [ベンダーモジュールと連携してコンソールから設定を投入する場合](./playbook-example/cat3550_config.md)

* Cisco IOSと連携してコンソールからCisco装置に設定を投入するPlaybookの例となります。

<br>
<br>

## 4. [ベンダーモジュールと連携してコンソールから情報を取得する場合](./playbook-example/cat3550_show.md)

* Cisco IOSと連携してコンソールからCisco装置の情報を取得するPlaybookの例となります。

<br>
<br>

## 5. [コンソールからターゲット装置のバージョンアップを実行する場合](./playbook-example/cat1000_verup.md)

* `smartcs_tty_command`を使用して、コンソールからCisco装置のバージョンアップを実行するPlaybookの例となります。

<br>
<br>

[↑トップページに戻る](../README.md)
