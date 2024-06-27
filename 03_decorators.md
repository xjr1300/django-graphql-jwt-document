# デコレーター

<https://django-graphql-jwt.domake.io/decorators.html>

- [デコレーター](#デコレーター)
  - [@login\_required](#login_required)
  - [@user\_passes\_test](#user_passes_test)
  - [@permission\_required](#permission_required)
  - [@staff\_member\_required](#staff_member_required)
  - [@superuser\_required](#superuser_required)

## @login_required

```text
login_required = <function user_passes_test.<locals>.decorator>
```

短縮して、`login_required()`デコレーターを使用できます。

```python
import graphene
from graphql_jwt.decorators import login_required


class Query(graphene.ObjectType):
    viewer = graphene.Field(UserType)

    @login_required
    def resolve_viewer(self, info, **kwargs):
        return info.context.user
```

- ユーザーがログインしていない場合、`PermissionDenied`例外が発生します。
- ユーザーがログインしている場合、普通に関数を実行します。

## @user_passes_test

```text
user_passes_test(test_func, exc=<class 'graphql_jwt.exceptions.PermissionDenied'>)
```

短縮して、呼び出し可能なオブジェクトが`False`を返したときに、`PermissionDenied`例外を発生する、便利な`user_passes_test()`デコレーターを使用できます。

```python
from django.contrib.auth import get_user_model

import graphene
from graphql_jwt.decorators import user_passes_test


class Query(graphene.ObjectType):
    users = graphene.List(UserType)

    @user_passes_test(lambda user: user.email.contains("@staff"))
    def resolve_users(self, info, **kwargs):
        return get_user_model().objects.all()
```

`user_passes_test()`は、必須の引数を受け取ります。
`User`オブジェクトを受け取り、ユーザーが行動を実行することを許可されている場合に`True`を返す呼び出し可能オブジェクトを受け取ります。
`user_passes_test()`は、自動的に`User`がアノニマスでないか確認したいことに注意してください。

## @permission_required

```text
permission_required(perm)
```

ユーザーが特定のパーミッションを持っているかどうか確認するデコレーターです。
ちょうど`has_perm()`メソッドのように、パーミッション名の書式を受け取ります。

```text
<app-label>.<permission-codename>
```

また、そのデコレーターは、パーミッションのイテラブルを受け付けるかもしれず、その場合、リゾルバーまたはミューテーションにアクセスするために、ユーザーはすべてのパーミッションを持たなければなりません。

```python
import graphene
from graphql_jwt.decorators import permission_required


class UpdateUser(graphene.Mutation):
    class Arguments:
        user_id = graphene.Int()

    @classmethod
    @permission_required("auth.change_user")
    def mutate(cls, root, info, user_id):
        # ...
```

## @staff_member_required

```text
staff_member_required = <function user_pass_test.<locals>.decorator>
```

この関数で就職されたリゾルバーまたはミューテーションは、次の振る舞いを持ちます。

ユーザーが、`User.is_staff=True`でスタッフメンバーの場合、普通に関数を実行します。
そうでない場合、`PermissionDenied`例外が発生します。

```python
from django.contrib.auth import get_user_model

import graphene
from graphql_jwt.decorators import staff_member_required


class Query(graphene.ObjectType):
    users = graphene.List(UserType)

    @staff_member_required
    def resolve_users(self, info, **kwargs):
        return get_user_model().objects.all()
```

## @superuser_required

```text
superuser_required = <function user_passes_test.<locals>.decorator>
```

この関数で就職されたリゾルバーまたはミューテーションは、次の振る舞いを持ちます。

ユーザーが`User.is_superuser=True`でスーパーユーザーの場合、普通に関数を実行します。
そうでない場合、`PermissionDenied`例外が発生します。

```python
import graphene
from graphql_jwt.decorators import superuser_required


class DeleteUser(graphene.Mutation):
    class Arguments:
        user_id = graphene.Int()

    @classmethod
    @superuser_required
    def mutate(cls, root, info, user_id):
        # ...
```
