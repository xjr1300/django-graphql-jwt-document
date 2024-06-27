# 認証

<https://django-graphql-jwt.domake.io/authentication.html>

- [認証](#認証)
  - [HTTPヘッダー](#httpヘッダー)
  - [クッキーごとに](#クッキーごとに)
    - [クッキーの削除](#クッキーの削除)
  - [引数ごとに](#引数ごとに)
    - [設定](#設定)
    - [スキーマ](#スキーマ)
    - [クエリ](#クエリ)

*Django-graphql-jwt`は、認証されたユーザーを*コンテキスト*オブジェクトにいれるフックをするために、[Grapheneミドルウェア](https://docs.graphene-python.org/en/latest/execution/middleware/)を使用します。
データへのアクセスを制限する単純で、そのままの方法は、`info.context.user.is_authenticated`を確認することです。

```python
import graphene


class Query(graphene.ObjectType):
    viewer = graphene.Field(graphene.UserType)

    def resolve_viewer(self, info, **kwargs):
        user = info.context.user
        if not user.is_authenticated:
            raise Exception("Authentication credentials were not provided")
        return user
```

短縮するために、*リゾルバー*と*ミューテーション*に[デコレーター](https://django-graphql-jwt.domake.io/decorators.html)を使用することができます。

## HTTPヘッダー

これで、保護されたAPIにアクセス刷るために、`Authorization`HTTPヘッダーを含めなくては成りません。

```http
POST / HTTP/1.1
Host: domake.io
Authorization: JWT <token>
Content-Type: application/json;
```

## クッキーごとに

トークンが要求され、`jwt_cookie`デコレーターが設定されたとき、レスポンスはトークン文字列を持つクッキーが設定されます。

```python
from django.urls import path

from graphene_django.views import GraphQLView
from graphql_jwt.decorators import jwt_cookie


urlpatterns = [
    path("graphql/", jwt_cookie(GraphQLView.as_view())),
]
```

もし、`jwt_cookie`デコレーターが設定された場合、[クロスサイトリクエストフォージェリ](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))に対する保護を提供するために、[CSRFミドルウェア](https://docs.djangoproject.com/es/2.1/ref/csrf/)である`django.middleware.csrf.CsrfViewMiddleware`を追加することを検討してください。

クッキーを基盤とした認証尾は、ミューテーションの入力引数としてトークンを送信する必要がなくなります。

### クッキーの削除

XSS（クロスサイトスクリプティング）攻撃を回避刷るために、クッキーは`HttpOnly`フラグが設定されているため、クライアントサイドでそれらを削除できません。
このパッケージは、サーバーサイドでクッキーを削除するためのいくつかのミューテーションを含んでいます。

ルートスキーマにミューテーションを追加します。

```python
import graphene
import graphql_jwt


class Mutation(graphene.ObjectType):
    delete_token_cookie = graphql_jwt.DeleteJSONWebTokenCookie.Field()

    # 長期間有効なリフレッシュトークン
    delete_refresh_token_cookie = graphql_jwt.DeleteRefreshTokenCookie.Field()


schema = graphene.Schema(mutation=Mutation)
```

- `JWT`クッキーを削除するために`deleteTokenCookie`を呼び出してください。

```graphql
mutation {
  deleteTokenCookie {
    deleted
  }
}
```

- [長期間有効なリフレッシュトークン](https://django-graphql-jwt.domake.io/refresh_token.html)のための`JWT-refresh-token`クッキーを削除するために`deleteRefreshTokenCookie`を呼び出してください。

## 引数ごとに

*トークン*を送信するほかの選択肢は、*GraphQL*クエリ内の引数を使用することで、それは異なるクレデンシャルで認証されたクエリの束を送信することを可能にします。

*Django-graphql-jwt`は、送信された引数のリスト内から*トークン*を探して、もしそれが存在しない場合、HTTPヘッダー内からトークンを探します。

### 設定

次の通り`settings`で引数認証を有効にします。

```python
GRAPHQL_JWT = {
    "JWT_ALLOW_ARGUMENT": True,
}
```

### スキーマ

`JWT_ARGUMENT_NAME`設定で定義された同じ名前を使用する任意のフィールドに、*トークン*引数を追加します。

```python
import graphene
from graphql_jwt.decorators import login_required


class Query(graphene.ObjectType):
    viewer = graphene.Field(UserType, token=graphene.String(required=True))

    @login_required
    def resolve_viewer(self, info, **kwargs):
        return info.context.user
```

### クエリ

クエリ内で他の変数としてトークンを送信します。

```graphql
query GetViewer($token: String!) {
  viewer(token: $token) {
    username
    email
  }
}
```

**複数のクレデンシャル**を使用して認証します。

```graphql
query GetUsers($tokenA: String!, $tokenB: String!) {
  viewerA: viewer(token: $tokenA) {
    username
    email
  }
  viewerB: viewer(token: $tokenB) {
    username
    email
  }
}
```
