# シグナル

<https://django-graphql-jwt.domake.io/signals.html>

- [シグナル](#シグナル)
  - [token\_issued](#token_issued)
  - [token\_refreshed](#token_refreshed)
  - [refresh\_token\_rotated](#refresh_token_rotated)
  - [refresh\_token\_revoked](#refresh_token_revoked)

`django-graphql-jwt`は次のシグナルを使用します。

## token_issued

ユーザーが認証に成功したときに送信されます。
このシグナルで送信される引数は次のとおりです。

- sender: Grapheneのミューテーションのクラス
- request: 現在のHttpRequestインスタンス
- user: ちょうど認証されたユーザーインスタンス

## token_refreshed

シングルトークンがリフレッシュされたときに送信されます。

このシグナルで送信される引数は次のとおりです。

- sender: Grapheneのミューテーションクラス
- request: 現在のHttpRequestインスタンス
- user: ちょうどシングルトークンをリフレッシュしたユーザーインスタンス

## refresh_token_rotated

有効期限の長いリフレッシュトークンがローテーションされたときに送信されます。

このシグナルで送信される引数は次のとおりです。

- sender: ちょうどローテーションされたリフレッシュトークンのクラス
- request: 現在のHttpRequestインスタンス
- refresh_token: ちょうどローテーションされた古いリフレッシュトークンインスタンス
- refresh_token_issued: 発行された新しいリフレッシュトークンインスタンス

## refresh_token_revoked

有効期限の長いリフレッシュトークンが無効になったときに送信されます。

このシグナルで送信される引数は次のとおりです。

- sender: ちょうど無効になったリフレッシュトークンのクラス
- request: 現在のHttpRequestインスタンス
- refresh_token: ちょうど無効にされたリフレッシュトークンインスタンス
