# Auth Service

[Source code](https://github.com/nildev/auth) at GitHub.

Service provides authentication and authorization. It has built in integration with:

* Github

Service exposes `auth` endpoint which accepts `auth_code` from given `provider` and on successful authentication returns [JWT](https://jwt.io/) token. You can pass this token to your other services to authenticate and authorize API requests.