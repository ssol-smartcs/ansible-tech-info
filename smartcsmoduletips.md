# 本ページはメンテナンス中です。

[↑目次に戻る](./README.md)
<br>
# トラブルシューティング (SmartCS x Ansible)

本ページでは、Ansible のエラーメッセージが出力されず、SmartCS x Ansible の連携で期待する実行結果とならない場合の対処を説明しています。  
実行結果、想定される原因および対処方法について記載しています。  
Playbook 実行時にAnsible のエラーメッセージが出力される場合の対処方法は、[トラブルシューティング（Ansible）](./troubleshooting.md)を参照してください。  

`注意事項`  
記載されている対処方法により期待するオペレーションを必ず実現できるとは限りません。  
実現のための参考情報としてご活用いただけますと幸いです。  

<br>

## 目次
- [1. sendchar で指定した文字列が想定通りに実行されません。](./smartcsmoduletips.md#1-sendchar-で指定した文字列が想定通りに実行されません)

<br>
<br>

## 1. sendchar で指定した文字列が想定通りに実行されません。
#### 想定原因
`recvchar` に指定する文字列に不足/誤りがある、あるいはターゲット機器のコンソールが既にログイン後の状態となっているなど、Playbook 開始時点で想定された状態となっていない可能性があります。  

#### 対処方法
- `recvchar` に指定する文字列に不足/誤りがある場合  
`sendchar` で指定したコマンドを実行した際に、ターゲット機器のコンソールから出力される文字列を再度ご確認ください。
また、`recvchar` で指定する文字列に不要な半角スペースなどが含まれている場合、受信した文字列が`recvchar` に含まれていないと判断されます。  
受信する文字列が`recvchar` で正しく指定されているかどうか、併せてご確認ください。

- Playbook 開始時点で想定された状態となっていない場合  
コンソール経由での操作となるため、Playbook 実行時のコンソールの状態は、ログイン前、ログイン後、コンフィグレーションモード移行後など、前回の操作内容により様々なケースが想定されます。  
`smartcs_tty_command` モジュールを使用した Playbook を作成する際は、最後に`exit` コマンドなどを実行してログアウト状態で終了する様な構成にすることを推奨いたします。  
また、`smartcs_tty_command` モジュールには、`sendchar` の送信を開始する前にコンソールの状態をチェックする機能(プレチェック機能)があります。Playbook 開始時に想定したコンソール状態になっていない場合、`sendchar` を送信せずに終了する、コンソールを期待した状態に戻すために`exit` コマンドを実行する、などの動作を指定する事が可能です。  



[↑目次に戻る](./README.md)