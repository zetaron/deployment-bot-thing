# deployment bot thing - GraphQL-API

## GraphQL Schema
```js
const typeDefinitions = `

# created by link to hoster account
type User {
  username: String!
  projects: [Project]
  deployments: [Deployment]
}

# parsed from config file inside repository
# cached to db
type Environment {
  name: String!
  project: Project!
  color: String!
  description: String
}

type Project {
  name: String! # name = "/:owner.username/:project"
  owner: User!
  environments: [Environment]
  deployments: [Deployment]
}

type Deployment {
  id: Int!
  issuer: User!
  timestamp: Int!
  status: String!
  environment: String!
  commitId: String!
  
  task: String
  containerId: String
}

# the schema allows the following two queries:
type RootQuery {
  deployment(id: Int): Deployment
  user(username: String): User
  project(projectname: String): Project
}

# this schema allows the following two mutations:
type RootMutation {
  registerUser(
    username: String
  ): User
  
  createProject(
    owner: String
    name: String
  ): Project
  
  createDeployment(
    project: String
    environment: String
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
