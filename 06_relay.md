# Relay

<https://django-graphql-jwt.domake.io/relay.html>

- [Relay](#relay)
  - [スキーマ](#スキーマ)
  - [クエリ](#クエリ)
  - [シングルトークンリフレッシュ](#シングルトークンリフレッシュ)
  - [有効期限の長いリフレッシュトークン](#有効期限の長いリフレッシュトークン)
  - [クッキー](#クッキー)
  - [カスタマイズ](#カスタマイズ)

[Relay](https://facebook.github.io/relay/)を完全にサポートしています。

## スキーマ

ルートスキーマにミューテーションを追加します。

```python
import graphene
import graphql_jwt


class Mutation(graphene.ObjectType):
    token_auth = graphql_jwt.relay.ObtainJSONWebToken.Field()
    verify_token = graphql_jwt.relay.Verify.Field()
    refresh_token = graphql_jwt.relay.Refresh.Field()
    delete_token_cookie = graphql_jwt.relay.DeleteJSONWebTokenCookie.Field()

    # 長期間有効なリフレッシュトークン
    revoke_token = graphql_jwt.relay.Revoke.Field()

    delete_refresh_token_cookie = \
        graphql_jwt.relay.DeleteRefreshTokenCookie.Field()


schema = graphene.Schema(mutation=Mutation)
```

## クエリ

Relayのミューテーションは、*input*と名付けられた単一の引数を受け取ります。

- ユーザーを認証して、**JSON Webトークン**を取得するために`tokenAuth`を呼び出します。

```graphql
mutation TokenAuth($username: $String!, $password: String!) {
  tokenAuth(input: {username: $username, password: $password}) {
    token
    payload
    refreshExpiresIn
  }
}
```

- *トークン*を検証して*トークンのペイロード*を取得するために`verifyToken`を呼び出します。

```graphql
mutation VerifyToken($token: String!) {
  verifyToken(input: {token: $token}) {
    payload
  }
}
```

## シングルトークンリフレッシュ

- **有効期限が切れていないトークン**で更新されたゆうっこう期限を持つ新しい*トークン*を取得するために`refreshToken`を呼び出します。

```graphql
mutation RefreshToken($token: String!) {
  refreshToken(input: {token: $token}) {
    token
    payload
    refreshExpiresIn
  }
}
```

## 有効期限の長いリフレッシュトークン

- 認証時にすでに取得したり**リフレッシュトークン**を使用して、*トークン*をリフレッシュ刷るために`refreshToken`を呼び出します。

```graphql
mutation RefreshToken($refreshToken: String!) {
  refreshToken(input: {refreshToken: $refreshToken}) {
    token
    payload
    refreshToken
    refreshExpiresIn
  }
}
```

- 有効な**リフレッシュトークン**を無効にするために`revokeToken`を呼び出します。
  無効化はすぐに行われるため、その**リフレッシュトークン**は、無効になった後で、再度使用できません。

```graphql
mutation RevokeToken($refreshToken: String!) {
  revokeToken(input: {refreshToken: $refreshToken}) {
    revoked
  }
}
```

## クッキー

- `JWT`クッキーを削除するために`deleteTokenCookie`を呼び出してください。

```graphql
mutation {
  deleteTokenCookie(input: {}) {
    deleted
  }
}
```

- [長期間有効なリフレッシュトークン](https://django-graphql-jwt.domake.io/refresh_token.html)用に`JWTリフレッシュトークン`クッキーを削除するために`deleteRefreshTokenCookie`を呼び出してください。

```graphql
mutation {
  deleteRefreshTokenCookie(input: {}) {
    deleted
  }
}
```

## カスタマイズ

`ObtainJSONWebToken`の振る舞いをカスタマイズしたい場合、サブクラスの`resolve()`メソッドをカスタマイズする必要があります。

```text
class JSONWebTokenMutation
```

```python
import graphene
import graphql.relay


class ObtainJSONWebToken(relay.JSONWebTokenMutation):
    user = graphene.Field(UserType)

    @classmethod
    def resolve(cls, root, info, **kwargs):
        return cls(user=info.context.user)
```

ユーザーを認証して、**JSON Webトークン**と*ユーザーID*を取得します。

```graphql
mutation TokenAuth($username: String!, $password: String!) {
  tokenAuth(input: {username: $username, password: $password}) {
    token
    payload
    refreshExpiresIn
    user {
      id
    }
  }
}
```
