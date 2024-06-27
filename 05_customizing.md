# カスタマイズ

<https://django-graphql-jwt.domake.io/customizing.html>

- [カスタマイズ](#カスタマイズ)

`ObtainJSONWebToken`の振る舞いをカスタマイズしたい場合、サブクラスの`resolve()`メソッドをカスタマイズ刷る必要があります。

```text
class JSONWebTokenMutation
```

```python
import graphene
import graphql_jwt


class ObtainJSONWebToken(graphql_jwt.JSONWebTokenMutation):
    user = graphene.Field(UserType)

    @classmethod
    def resolve(cls, root, info, **kwargs):
        return cls(user=info.context.user)
```

ユーザーを認証して、**JSON Webトークン**と*ユーザーID*を取得します。

```graphql
mutation TokenAuth($username: String!, $password: String!) {
  tokenAuth(username: $username, password: $password) {
    token
    payload
    user {
      id
    }
  }
}
```
