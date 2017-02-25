# deployment bot thing - GraphQL-API

## Architecture

```
+----------------------------+                   +-------------------+    +---------------------+
|                            |                   |                   |    |                     |
| External Service           |  deployment hook  | Provider API      |    | API/Relay API       |
|                            +-------------------> * map deployment  +----> * create deployment |
+------------------+---------+                   |                   |    |                     |
                   |                             +-------------------+    +-----+---------------+
                   |                                                            |
                   |                                                            |
                   |                                                            | send deployment
                   |                                                            |
                   |                                                            |
                   |                                        +-------------------v---------------+
                   |  git clone/pull/fetch                  |                                   |
                   +----------------------------------------> Deployment Runner                 |
                                                            | * has access to git (via secrets) |
                         +---------------------+            | * has access to docker registry   |
                         |                     |            | * knows swarm master to deploy to |
                         | Docker Swarm Master |  schedule  | * is allowed to schedule services |
                         |                     <------------+                                   |
                         +---------------------+            +-----------------------------------+
```
Graph drawn using: [http://asciiflow.com/](http://asciiflow.com/)

## MVP API
### OpenAPI spec
See the `swagger/swagger.yaml` (View/Edit using [Swagger Editor](http://editor.swagger.io/))

### Tools
- [go-openapi](https://github.com/go-openapi)
- 
