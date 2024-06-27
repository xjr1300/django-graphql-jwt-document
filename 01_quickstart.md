# クイックスタート

<https://django-graphql-jwt.domake.io/quickstart.html>

- [クイックスタート](#クイックスタート)
  - [依存関係](#依存関係)
  - [インストール](#インストール)
  - [スキーマ](#スキーマ)
  - [クエリ](#クエリ)

## 依存関係

- Python 3.6+
- Django 2.0+

## インストール

Pypiから最新の安定版のv0.4.0をインストールしてください。

```sh
pip install django-graphql-jwt
```

`MIDDLEWARE`設定に`AuthenticationMiddleware`ミドルウェアを追加してください。

```python
MIDDLEWARE = [
    ...
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    ...
]
```

`GRAPHENE`設定に`JSONWebTokenMiddleware`ミドルウェアを追加してください。

```python
GRAPHENE = {
    "SCHEMA": "mysite.myschema.schema",
    "MIDDLEWARE": [
        "graphql_jwt.middleware.JSONWebTokenMiddleware",
    ],
}
```

`AUTHENTICATION_BACKENDS`に`JSONWebTokenBackend`バックエンドを追加してください。

```python
AUTHENTICATION_BACKENDS = [
    "graphql_jwt.backends.JSONWebTokenBackend",
    "django.contrib.auth.backends.ModelBackend",
]
```

## スキーマ

ルートスキーマにミューテーションを追加してください。

```python
import graphene
import graphql_jwt


class Mutation(graphene.ObjectType):
    token_auth = graphql_jwt.ObtainJSONWebToken.Field()
    verify_token = graphql_jwt.Verify.Field()
    refresh_token = graphql_jwt.Refresh.Field()


schema = graphene.Schema(mutation=Mutation)
```

## クエリ

- ユーザーを認証して、**JSON Web Token**を取得するために`tokenAuth`を呼び出してください。

ミューテーションは、ユーザーモデルの[USERNAME_FIELD](https://docs.djangoproject.com/en/2.0/topics/auth/customizing/#django.contrib.auth.models.CustomUser)を使用して、デフォルトでそれは`username`です。

```graphql
mutation TokenAuth($username: String!, $password: String!) {
  tokenAuth(username: $username, password: $password) {
    token
    payload
    refreshExpiresIn
  }
}
```

- *トークン*を検証して、*トークンのペイロード*を取得するために`verifyToken`を呼び出してください。

```graphql
mutation VerifyToken($token: String!) {
  verifyToken(token: $token) {
    payload
  }
}
```

- 再更新された有効期限を持つ新しい*トークン*を取得するために`refreshToken`を呼び出してください。

[リフレッシュトークンのシナリオを構成](https://django-graphql-jwt.domake.io/refresh_token.html)して、[JWT_VERIFY_EXPIRATION](https://django-graphql-jwt.domake.io/settings.html)設定を`True`に設定してください。
