# spicedb-python

An unofficial SpiceDB client for Python.

```
pip install spicedb
```

Note that this is a very unstable package and the API is subject to change.

Source code: https://codeberg.org/aedge/spicedb-python


## Getting started

Initialize the client:

```py
from spicedb import SpiceDB, SpiceRelationship

client = SpiceDB("https://api.spicedb.example.net/", "API_KEY")
```

Set up a schema:

```py
await client.write_schema("""
    definition user {}

    definition team {
        relation admin: user
        relation member: user
    }

    definition project {
        relation parent: team

        permission view = parent->member
        permission edit = parent->admin
    }
""")
```

Create some relationships:

```py
alice = {"type": "user", "id": "alice"}
bob = {"type": "user", "id": "bob"}
team = {"type": "team", "id": "acme-co"}
cool_project = {"type": "project", "id": "cool_project"}

await client.bulk(
    create=[
        SpiceRelationship(team, "parent", cool_project),
        SpiceRelationship(alice, "admin", team),
        SpiceRelationship(bob, "member", team),
    ],
)
```

Check permissions:

```py
await client.authorize(alice, "edit", cool_project)  # => True
await client.authorize(bob, "edit", cool_project)    # => False
```

List resources:

```py
await client.list(alice, "edit", "project")  # => ["cool_project"]
```
