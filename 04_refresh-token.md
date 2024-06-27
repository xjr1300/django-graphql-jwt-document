# トークンのリフレッシュ

<https://django-graphql-jwt.domake.io/refresh_token.html>

- [トークンのリフレッシュ](#トークンのリフレッシュ)
  - [シングルトークンリフレッシュ](#シングルトークンリフレッシュ)
    - [設定](#設定)
    - [クエリ](#クエリ)
      - [更新と生きているトークンの保持](#更新と生きているトークンの保持)
      - [4分後にリフレッシュする](#4分後にリフレッシュする)
  - [長期間有効なトークンのリフレッシュ](#長期間有効なトークンのリフレッシュ)
    - [設定](#設定-1)
    - [スキーマ](#スキーマ)
    - [クエリ](#クエリ-1)
    - [クッキーごとに](#クッキーごとに)
    - [無制限のリフレッシュ](#無制限のリフレッシュ)
    - [リフレッシュトークンの1回だけの使用](#リフレッシュトークンの1回だけの使用)
    - [リフレッシュトークンのクリア](#リフレッシュトークンのクリア)

このパッケージは、2つの更新方法をサポートしています。

- [シングルトークンリフレッシュ](https://django-graphql-jwt.domake.io/refresh_token.html#single-token-refresh)（デフォルト）
- [長期間有効なトークンのリフレッシュ](https://django-graphql-jwt.domake.io/refresh_token.html#long-running-refresh-tokens)

## シングルトークンリフレッシュ

### 設定

```python
GRAPHQL_JWT = {
    "JWT_VERIFY_EXPIRATION": True,
    "JWT_EXPIRATION_DELTA": timedelta(minutes=5),
    "JWT_REFRESH_EXPIRATION_DELTA": timedelta(days=7)<>
}
```

それは、5分ごとに更新する必要があり(`payload.exp`)、5分ごとにトークンを更新し続けても、最初のトークンが発行されてから7日後にログアウトされる(`refreshExpiresIn`)ことを意味します。

### クエリ

- **有効期限が切れていないトークン**で、更新された有効期限を持つ新しい*トークン*を得るために`refreshToken`を呼び出します。

```graphql
mutation RefreshToken($token: String!) {
  refreshToken(token: $token) {
    token
    payload
    refreshExpiresIn
  }
}
```

#### 更新と生きているトークンの保持

![refresh and keeping tokens alive](https://user-images.githubusercontent.com/5514990/34951332-e67845f0-fa3b-11e7-8e72-09d610e73025.png)

- トークンが発行されてから5分経過すると、トークンが無効になる。
  - `verifyToken`を呼び出すとエラーが発生する。
- ただし、7日経過するまでそのトークンで新しいトークンを歳取得できる。

#### 4分後にリフレッシュする

![refresh after 4 minutes](https://user-images.githubusercontent.com/5514990/34951331-e2ff9680-fa3b-11e7-8f0a-dbb3845367a7.png)

- 4分後にトークンをリフレッシュすると、後5分、最初にトークンを取得してから9分後まで、そのトークンは有効である。
- ただし、そのトークンのリフレッシュは、最初にトークンを取得してから7日後までしかできない。

- トークンを発行

```text
exp = orig_iat + JWT_EXPIRATION_DELTA (payload.exp)
refreshToken (t): exp = t + JWT_EXPIRATION_DELTA
```

- 署名の有効期限（ログインが要求される）

```text
when: t = exp
exp = refresh_at + JWT_EXPIRATION_DELTA
verifyToken (t): error! if JWT_VERIFY_EXPIRATION=true
```

- リフレッシュの有効期限

```text
when: t = orig_iat + JWT_REFRESH_EXPIRATION_DELTA (refreshExpiresIn)
refreshToken (t): error!
```

## 長期間有効なトークンのリフレッシュ

データベースに保存されたリフレッシュトークンです。

`INSTALLED_APPS`に`graphql_jwt.refresh_token`を追加します。

```python
INSTALLED_APPS = [
    # ...
    "graphql_jwt.refresh_token.apps.RefreshTokenConfig",
    # ...
]
```

### 設定

```python
GRAPHQL_JWT = {
    "JWT_VERIFY_EXPIRATION": True,
    "JWT_LONG_RUNNING_REFRESH_TOKEN": True,
    "JWT_EXPIRATION_DELTA": timedelta(minutes=5),
    "JWT_REFRESH_EXPIRATION_DELTA": timedelta(days=7),
}
```

それは、5分ごとにリフレッシュ(`payload.exp`)が必要で、リフレッシュトークンが発行されてから7日後(`refreshExpiresIn`)にリフレッシュトークンを置き換える必要があることを示します。

### スキーマ

ルートスキーマにミューテーションを追加します。

```python
import graphene
import graphql_jwt


class Mutation(graphene.ObjectType):
    token_auth = graphql_jwt.ObtainJSONWebToken.Field()
    verify_token = graphql_jwt.Verify.Field()
    refresh_token = graphql_jwt.Refresh.Field()
    revoke_token = graphql_jwt.Revoke.Field()


schema = graphene.Schema(mutation=Mutation)
```

### クエリ

- ユーザーを認証して、**JSON Webトークン**と**リフレッシュトークン**を取得するために`tokenAuth`を呼び出します。

```graphql
mutation TokenAuth($username: String!, $password: String!) {
  tokenAuth(username: $username, password: $password) {
    token
    payload
    refreshToken
    refreshExpiresIn
  }
}
```

- 認証時にすでに取得した**リフレッシュトークン**を使用して、*トークン*をリフレッシュするために`refreshToken`を呼び出します。

```graphql
mutation RefreshToken($refreshToken: String!) {
  refreshToken(refreshToken: $refreshToken) {
    token
    payload
    refreshToken
    refreshExpiresIn
  }
}
```

- 有効な*リフレッシュトークン*を無効にするために`revokeToken`を呼び出します。
  無効化はすぐに実施され、執行した後でリフレッシュトークンを再び使用することはできません。

```graphql
mutation RevokeToken($refreshToken: String!) {
  revokeToken(refreshToken: $refreshToken) {
    revoked
  }
}
```

### クッキーごとに

リフレッシュトークンが要求され、`jwt_cookie`デコレーターが設定されている場合、レスポンスは、リフレッシュトークン文字列を持つクッキーが与えられます。

### 無制限のリフレッシュ

リフレッシュトークンが期限切れか確認する`JWT_REFRESH_EXPIRATION_HANDLER`設定を構成してください。

```python
GRAPHQL_JWT = {
    "JWT_VERIFY_EXPIRATION": True,
    "JWT_LONG_RUNNING_REFRESH_TOKEN": True,
    "JWT_REFRESH_EXPIRED_HANDLER": lambda orig_iat, context: False,
}
```

### リフレッシュトークンの1回だけの使用

リフレッシュトークンが使用された後、リフレッシュトークンを自動的に無効にできます。

```python
from django.dispatch import receiver

from graphql_jwt.refresh_token.signals import refresh_token_rotated


@receiver(refresh_token_rotated)
def revoke_refresh_token(sender, request, refresh_token, **kwargs):
    refresh_token.revoke(request)
```

### リフレッシュトークンのクリア

```text
Command.handle(expired, *args, **options)
コマンドの実際のロジックです。サブクラスはこのメソッドを実装する必要があります。
```

`cleartokens`コマンドで、無効になったリフレッシュトークンを削除します。

```sh
$ python manage.py cleartokens --help

usage: cleartokens [--expired]

optional arguments:
  --expired             Clears expired tokens
```

`--expired`引数は、`JWT_REFRESH_EXPIRATION_DELTA`設定によって指定された値よりも、生存期間が長いリフレッシュトークンを削除します。
