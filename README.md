# deployment bot thing - GraphQL-API

## Architecture

```
+------------------+                              +---------------------+
|                  |                              |                     |
| GIT              |    on deployment hook        | API                 |      read
| * config         +------------------------------> * create deployment <---------------+
|   * environments |                              | * watch deployment  |               |
|   * trigger      |                              |                     |               |
| * scripts to run |                              +-----+---------------+               |
|                  |                                    |                               |
+------------------+                                    |                        +------+-------+
                                                        |                        |              |
                                                        |                        |  ELK Logging |
                                                        |start/watch             |              |
                                                        |                        +------^-------+
                                                        |                               |
                                                        |                               |
                                                        |                               |
                                                  +-----v---------------------------+   |write
                                                  |                                 |   |
                                                  | Deployment Runner               |   |
                                                  | * has access to git             |   |
                                                  | * has access to build artifacts +---+
                                                  | * runs deployment scripts       |
                                                  | * knows hosts to deploy to      |
                                                  |                                 |
                                                  +---------------------------------+
```

## GraphQL Schema
```js
const typeDefinitions = `

type Project {
  name: String! # name = ":git-host/:owner.username/:project"
  deployments: [Deployment]
}

type Deployment {
  id: Int!
  timestamp: Int!
  status: String!
  environment: String!
  commitId: String!
  project: Project!
  
  task: String
}

# the schema allows the following two queries:
type RootQuery {
  deployment(projectname: String!, id: Int!): Deployment
  project(projectname: String!): Project
}

# this schema allows the following mutations:
type RootMutation {
  createDeployment(
    project: String! # name = ":git-host/:owner.username/:project"
    commitId: String! 
    environment: String! 
    task: String
  ): Deployment
}

# we need to tell the server which types represent the root query
# and root mutation types. We call them RootQuery and RootMutation by convention.

schema {
  query: RootQuery
  mutation: RootMutation
}
`;

export default [typeDefinitions];
```
