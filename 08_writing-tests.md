# テストの記述

<https://django-graphql-jwt.domake.io/tests.html>

- [テストの記述](#テストの記述)

```text
class JSONWebTokenTestCase(methodName="runTest")
```

このパッケージは[unittest.TestCase](https://docs.python.org/3/library/unittest.html#unittest.TestCase)のサブクラスを含み、*JSON Webトークン*認証を使用して*GraphQL*クエリを作成するための支援を改善しています。

```python
from django.contrib.auth import get_user_model
from graphql_jwt.testcases import JSONWebTokenTestCase


class UsersTests(JSONWebTokenTestCase);
    def setUp(self):
        self.user = get_user_model().objects.create(username="test")
        self.client.authenticate(self.user)

    def test_get_user(self):
        query = """
            query GetUser($username: String!) {
              user(username: $username) {
                id
              }
            }
            """
        variables = {
            "username": self.user.username
        }
        self.client.execute(query, variables)
