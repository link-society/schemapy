Schemapy
========

*Schemapy* allows you to generate objects for centralized database access. You
define the schema for your API, the code that needs to be executed, then the
validation part is dealt with for you. Everything is then encapsulated in a single
object.

*Schemapy* relies on *PyDAL* for schema definition but is not tied to it.

Usage
-----

Using *PyDAL*:

```
from schemapy import API, DAL, Field
from datetime import datetime, timedelta

db = DAL('sqlite:memory')
db.define_table(
    'users',
    Field('name', type='string', required=True),
    Field('created_on', type='date')
)

db.define_table(
    'posts',
    Field('subject', type='string', required=True),
    Field('author', type='reference users', required=True),
    Field('created_on', type='date'),
    Field('content', type='text', required=True)
)

api = API(db)

@api.as_action(
    type='read',
    request=[
        Field('begin', type='date', required=True),
        Field('end', type='date', required=True)
    ],
    response=db.posts
)
def select_posts_by_date(db, req, action):
    query = (db.posts.created_on >= req.begin) | (db.posts.created_on <= req.end)
    return db(query).select()

now = datetime.now()
result = api.select_posts_by_date(
    begin=now - timedelta(days=1),
    end=now
)
print(list(result))
```
