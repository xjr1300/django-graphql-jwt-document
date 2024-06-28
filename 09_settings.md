# 設定

<https://django-graphql-jwt.domake.io/settings.html>

- [設定](#設定)
  - [PyJWT](#pyjwt)
    - [JWT\_ALGORITHM](#jwt_algorithm)
    - [JWT\_AUDIENCE](#jwt_audience)
    - [JWT\_ISSUER](#jwt_issuer)
    - [JWT\_LEEWAY](#jwt_leeway)
    - [JWT\_SECRET\_KEY](#jwt_secret_key)
    - [JWT\_PUBLIC\_KEY](#jwt_public_key)
    - [JWT\_PRIVATE\_KEY](#jwt_private_key)
    - [JWT\_VERIFY](#jwt_verify)
    - [JWT\_ENCODE\_HANDLER](#jwt_encode_handler)
    - [JWT\_DECODE\_HANDLER](#jwt_decode_handler)
    - [JWT\_PAYLOAD\_HANDLER](#jwt_payload_handler)
    - [JWT\_GET\_USER\_BY\_NATURAL\_KEY\_HANDLER](#jwt_get_user_by_natural_key_handler)
  - [トークンの有効期限](#トークンの有効期限)
    - [JWT\_VERIFY\_EXPIRATION](#jwt_verify_expiration)
    - [JWT\_EXPIRATION\_DELTA](#jwt_expiration_delta)
  - [リフレッシュトークン](#リフレッシュトークン)
    - [JWT\_ALLOW\_REFRESH](#jwt_allow_refresh)
    - [JWT\_REFRESH\_EXPIRATION\_DELTA](#jwt_refresh_expiration_delta)
    - [JWT\_LONG\_RUNNING\_REFRESH\_TOKEN](#jwt_long_running_refresh_token)
    - [JWT\_REFRESH\_TOKEN\_MODEL](#jwt_refresh_token_model)
    - [JWT\_REFRESH\_TOKEN\_N\_BYTES](#jwt_refresh_token_n_bytes)
    - [JWT\_REUSE\_REFRESH\_TOKENS](#jwt_reuse_refresh_tokens)
    - [JWT\_REFRESH\_EXPIRED\_HANDLER](#jwt_refresh_expired_handler)
    - [JWT\_GET\_+REFRESH\_TOKEN\_HANDLER](#jwt_get_refresh_token_handler)
  - [権限](#権限)
    - [JWT\_ALLOW\_ANY\_HANDLER](#jwt_allow_any_handler)
    - [JWT\_ALLOW\_ANY\_CLASSES](#jwt_allow_any_classes)
  - [HTTPヘッダー](#httpヘッダー)
    - [JWT\_AUTH\_HEADER\_NAME](#jwt_auth_header_name)
    - [JWT\_AUTH\_HEADER\_PREFIX](#jwt_auth_header_prefix)
  - [引数ごとに](#引数ごとに)
    - [JWT\_ALLOW\_ARGUMENT](#jwt_allow_argument)
    - [JWT\_ARGUMENT\_NAME](#jwt_argument_name)
  - [クッキー認証](#クッキー認証)
    - [JWT\_COOKIE\_NAME](#jwt_cookie_name)
    - [JWT\_REFRESH\_TOKEN\_COOKIE\_NAME](#jwt_refresh_token_cookie_name)
    - [JWT\_COOKIE\_SECURE](#jwt_cookie_secure)
    - [JWT\_COOKIE\_PATH](#jwt_cookie_path)
    - [JWT\_COOKIE\_DOMAIN](#jwt_cookie_domain)
    - [JWT\_COOKIE\_SAMESITE](#jwt_cookie_samesite)
    - [JWT\_HIDE\_TOKEN\_FIELDS](#jwt_hide_token_fields)
  - [CSRF](#csrf)
    - [JWT\_CSRF\_ROTATION](#jwt_csrf_rotation)
  - [推奨される設定](#推奨される設定)

*Django-graphql-jwt`は、`GRAPHQL_JWT`と名付けられた単独の**Django setting**から設定を読み込みます。

```python
GRAPHQL_JWT = {
    "JWT_VERIFY_EXPIRATION": True,
    "JWT_EXPIRATION_DELTA": timedelta(minutes=10),
}
```

ここに、*django-graphql-jwt*で利用できる**設定のリスト**と、それらのデフォルト値を示します。

## PyJWT

### [JWT_ALGORITHM](https://pyjwt.readthedocs.io/en/latest/algorithms.html)

暗号署名のアルゴリズムです。
デフォルトは、`"HS256"`です。

### [JWT_AUDIENCE](http://pyjwt.readthedocs.io/en/latest/usage.html#audience-claim-aud)

JWTが意図する受信者の識別子です。
デフォルトは、`None`です。

### [JWT_ISSUER](http://pyjwt.readthedocs.io/en/latest/usage.html#issuer-claim-iss)

JWTを発行したプリンシパルを識別します。
デフォルトは、`None`です。

### [JWT_LEEWAY](http://pyjwt.readthedocs.io/en/latest/usage.html?highlight=leeway#expiration-time-claim-exp)

過去ですが、あまり遠くない有効期限を検証します。
デフォルト値は、`timedelta(seconds=0)`です。

### [JWT_SECRET_KEY](https://pyjwt.readthedocs.io/en/latest/usage.html?highlight=secret%20key#usage-examples)

JWTを署名するときに使用される秘密鍵です。
デフォルトは、`settings.SECRET_KEY`です。

### [JWT_PUBLIC_KEY](https://django-graphql-jwt.domake.io/settings.html#jwt-public-key)

*RS256*、*RS384*、または*RS512*非対称アルゴリズムのRSAパブリックキーです。
`JWT_SECRET_KEY`設定は無視されます。
デフォルトは、`None`です。

### [JWT_PRIVATE_KEY](https://django-graphql-jwt.domake.io/settings.html#jwt-private-key)

*RS256*、*RS384*、または*RS512*非対称アルゴリズムのRSAプライベートキーです。
`JWT_SECRET_KEY`設定は無視されます。
デフォルトは、`None`です。

### [JWT_VERIFY](http://pyjwt.readthedocs.io/en/latest/usage.html?highlight=verify#reading-the-claimset-without-validation)

シークレットキーの検証です。
デフォルトは、`True`です。

### JWT_ENCODE_HANDLER

トークンをエンコードするカスタム関数です。
デフォルトは、`"graphql_jwt.utils.jwt_encode"`です。

### JWT_DECODE_HANDLER

トークンをデコードするカスタム関数です。
デフォルトは、`"graphql_jwt.utils.jwt_decode"`です。

### JWT_PAYLOAD_HANDLER

トークンのペイロードを生成するカスタム関数です。
デフォルトは、`"graphql_jwt.utils.jwt_payload"`です。

### JWT_GET_USER_BY_NATURAL_KEY_HANDLER

ユーザー名からUserオブジェクトを取得するカスタム関数です。
デフォルトは、`lambda payload: payload.get(get_user_model().USERNAME_FIELD)`です。

## トークンの有効期限

### [JWT_VERIFY_EXPIRATION](http://pyjwt.readthedocs.io/en/latest/usage.html?highlight=verify_exp#expiration-time-claim-exp)

検証の有効期限です。
デフォルトは、`False`です。

### JWT_EXPIRATION_DELTA

有効期限を設定するために、`utcnow()`に加えられる時間差分です。
デフォルトは、`timedelta(minutes=5)`です。

## リフレッシュトークン

### JWT_ALLOW_REFRESH

トークンのリフレッシュを有効化するかを示します。
デフォルトは、`True`です。

### JWT_REFRESH_EXPIRATION_DELTA

トークンをリフレッシュするときの期限です。
デフォルトは、`timedelta(days=7)`です。

### JWT_LONG_RUNNING_REFRESH_TOKEN

有効期限の長いリフレッシュトークンを有効にするかを示します。
デフォルトは、`False`です。

### JWT_REFRESH_TOKEN_MODEL

リフレッシュトークンを表現するために使用するモデルです。
デフォルトは、`"refresh_token.RefreshToken"`です。

### JWT_REFRESH_TOKEN_N_BYTES

有効期限の長いリフレッシュトークンのバイト数です。
デフォルトは、`20`です。

### JWT_REUSE_REFRESH_TOKENS

新しい有効期限の長いリフレッシュトークンを生成したが、データベースに存在するレコードを置き換えて、前の有効期限の長いリフレッシュトークンを無効にするかを示します。
デフォルトは、`False`です。

### JWT_REFRESH_EXPIRED_HANDLER

リフレッシュトークンが期限切れになったかを決定するカスタム関数です。
デフォルトは、`"graphql_jwt.utils.refresh_has_expired"`です。

### JWT_GET_+REFRESH_TOKEN_HANDLER

有効期限の長いリフレッシュトークンインスタンスを取得するカスタム関数です。
デフォルトは、`"graphql_jwt.refresh_token.utils.get_refresh_token_by_model"`です。

## 権限

### JWT_ALLOW_ANY_HANDLER

**フィールドごとに**非認証を決定するカスタム関数です。

> A custom function to determine the non-authentication per-field

デフォルトは、`"graphql_jwt.middleware.allow_any"`です。

### JWT_ALLOW_ANY_CLASSES

認証に必要としないGrapheneのクラスのリストまたはタプルです。
デフォルトは、`()`です。

## HTTPヘッダー

### JWT_AUTH_HEADER_NAME

認証ヘッダの名前です。
デフォルトは、`"HTTP_AUTHORIZATION"`です。

### JWT_AUTH_HEADER_PREFIX

認証ヘッダの接頭辞です。
デフォルトは、`"JWT"`です。

## 引数ごとに

### JWT_ALLOW_ARGUMENT

引数ごとに認証システムを許可します。
デフォルトは、`False`です。

### JWT_ARGUMENT_NAME

引数ごとの認証システムの引数名です。
デフォルトは、`"token"`です。

## クッキー認証

### JWT_COOKIE_NAME

トークンの妥当な転送として使用されるHTTPクッキーが使用されるときのクッキーの名前です。
デフォルトは、`"JWT"`です。

### JWT_REFRESH_TOKEN_COOKIE_NAME

リフレッシュトークンの妥当な転送として使用されるHTTPクッキーが使用されるときのクッキーの名前です。
デフォルトは、`"JWT-refresh-token"`です。

### JWT_COOKIE_SECURE

JTWクッキーのために安全なクッキーを使用するかどうかを示します。
これを`True`に設定した場合、クッキーは"secure"としてマークされ、それはブラウザがHTTPS接続のもとでのみそのクッキーを送信することを確実にします。

### JWT_COOKIE_PATH

クッキーのドキュメントの位置を示します。
デフォルトは、`"/"`です。

### JWT_COOKIE_DOMAIN

クロスドメインクッキーを設定する場合のドメイン名に使用します。
デフォルトは、`None`です。

### JWT_COOKIE_SAMESITE

クロスオリジンリクエストを実行するときに、JWTクッキーを送信しないことをブラウザに伝えるために、`'Strict'`または`'Lax'`を使用します。
Django バージョン2.1以降を要求します。

すべての同じサイトとクロスサイトリクエストでJWTクッキーを送信することを明示的に宣言するために`'None'`を使用します。

デフォルトは、`None`です。

### JWT_HIDE_TOKEN_FIELDS

クッキーを基盤とした認証のために、XSS搾取を避けるために、GraphQLスキーマからトークンフィールドを削除します。
デフォルトは、`False`です。

## CSRF

### JWT_CSRF_ROTATION

トークンまたはリフレッシュトークンが発行されるごとに、CSRFトークンをローテーションします。
デフォルトは、`False`です。

## 推奨される設定

- `jwt_cookie`デコレーターを使用して、ブラウザに対しては、クッキーを基盤とした認証を使用する。
- `jwt_cookie`デコレーターは、JWTに関するクッキーに[HttpOnlyフラグを設定](https://django-graphql-jwt.domake.io/authentication.html#delete-cookies)する。
- リフレッシュトークンを利用して、トークンが期限切れになった場合に、自動的にトークンを更新する。

```python
# リフレッシュトークンを有効化
JWT_LONG_RUNNING_REFRESH_TOKEN = True
# HTTPS接続のみでJWTクッキーを利用するようにブラウザに指示
JWT_COOKIE_SECURE = True
# 同じサイトにしかJWTクッキーを送信しないようにブラウザに指示
# Strictは、クロスサイトなサイトから、ログイン状態で遷移できないため、Laxを指定
JWT_COOKIE_SAMESITE = "Lax"
```
