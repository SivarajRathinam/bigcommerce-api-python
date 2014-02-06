Bigcommerce API V2 - Python Client
==================================

Lightweight wrapper over the `requests` library for communicating with the Bigcommerce v2 API.


Needs `requests` and `streql` (run `pip install bigcommerce-api` for easiest way to install),
and `nose` and `vcrpy` if you want to run the tests.

### Basic usage

of Connection:
```python
import bigcommerce as api  # imports Client, Connection, OAuthConnection, and HttpException classes

from pprint import pprint  # for nice output
```
```python
# connecting with basic auth and API key
HOST = 'www.example.com'
AUTH = ('username', 'apikey')

conn = api.Connection(HOST, AUTH)
pprint(conn.get('products', limit=5)  # supply any filter parameter as a keyword argument

try:
    p = conn.get('products', 35)
    print p.id, p.name  # p is a Mapping; a dict with . access to values
except ClientRequestException as e:
    if e.status_code == 404:
        print "failed to get product with id 35"
    print e.content

p = conn.update('products', p.id, {'name': 'Something Else'})
print p.id, p.name

imgs = conn.get('products/{}/images'.format(p.id))

# for deleting: conn.delete('resource', id)
# for posting: conn.create('resource', data)
```

and of OAuthConnection
```python
# after registering your app to get client id and secret
# and in your callback url handler, which should be passed code, context, and scope

conn = api.OAuthConnection(client_id, store_hash)  # store hash can be retrieved from context
# login_token_url is most likely "https://login.bigcommerceapp.com/oauth2/token"
token = conn.fetch_token(client_secret, code, context, scope, redirect_uri, login_token_url)
# conn can now be used like a Connection object to access resources


# if you already have the user's access token, simply do
conn = OAuthConnection(client_id, store_hash, access_token)

# and for constant-time verification of the signed payload passed to your load url
user_data = api.OAuthConnection.verify_payload(signed_payload, client_secret)  # returns False if authentication fails
```

### Exceptions

This library captures errors from the server in HttpException classes (included in the import `import bigcommerce`), which expose `status_code`, `headers`, and `content`. These exceptions will be raised for any non-200 status code (the exception to this is 204, which is raised for methods other than `Connection.delete`).

There are a few basic subclasses to HttpException: RedirectionException, ClientRequestException, ServerException, corresponding to 3xx, 4xx, and 5xx codes, and EmptyResponseWarning for 204.

If you find yourself wanting a more complete class heirarchy, or are otherwise aren't happy with the interface, please post an issue or otherwise contact me.