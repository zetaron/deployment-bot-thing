# deployment bot thing - GraphQL-API

## Architecture

```
+----------------------------+                       +-------------------+    +---------------------+
|                            |                       |                   |    |                     |
| GIT                        |  on deployment hook   | Provider API      |    | API/Relay API       |
| * docker-compose.stack.yml +-----------------------> * map deployment  +----> * create deployment |
|                            |                       |                   |    |                     |
+------------------+---------+                       +-------------------+    +-----+---------------+
                   |                                                                |
                   |                                                                |
                   |                                                                | send deployment
                   |                                                                |
                   |                                                                |
                   |                                              +-----------------v-----------------+
                   |        git clone/pull/fetch                  |                                   |
                   +----------------------------------------------> Deployment Runner                 |
                                                                  | * has access to git (via secrets) |
                         +---------------------+                  | * has access to docker registry   |
                         |                     |                  | * knows swarm master to deploy to |
                         | Docker Swarm Master |   schedule       | * is allowed to schedule services |
                         |                     <------------------+                                   |
                         +---------------------+                  +-----------------------------------+
```
Graph drawn using: [http://asciiflow.com/](http://asciiflow.com/)

## API
Gets deployed into the target cluster onto a master node.
Has an `environment` option which get added to each deployment request accepted by this API.

Discussion:
- maybe this API should only accept docker stacks or evaluated compose files, that way we could serve a broader band of providers aside from git.

### /deploy
#### POST Body:
```json
{
  "repository_url": "github.com/:user/:repo",
  "commit_sha": "<some-commits-sha1>"
}
```

### /deployments
#### GET
Fetches a list of previous deployments for which happened.
```json
[
  {
    // extends API /deployment POST Body
    "environment": "<environment>"
  }
]
```

##### Discussion:
- This endpoint should have some sort of filtering... eg. some sort of repo regex or timeframe or commit-sha

### /deployment/:id
#### GET
```json
{
  "id": "<deployment-id>",
  "original_request": {}, // what was received by the API
  "timestamp": "",
  "logs_url": "<this-url>/logs",
  "environment": "<environment>"
}
```

### /deployment/:id/logs
#### GET
RAW output of dockers log for this deployment.

#### HTTP/2 stream
RAW live-output of dockers log for this deployment.

## Environment Relay API
### /deploy
#### POST Body:
```json
{
  // extends API /deploy POST Body
  "environment": "<environment>" // the environment to relay this deployment to
}
```

### /deployments
#### GET
```json
[
  // aggregates all environments /deployments
]
```

### /deployment/:id
Relays the request to the correct environment.

### /deployment/:id/logs
Relays the request to the correct environment.

## (eg. GitHub) Provider API
### /github/:user/:repo/deploy
#### POST Body:
Some Provider (GitHub, Bitbucket, Travis, Docker-Cloud/Hub) specific payload which gets translated into `/deploy`

### /github/:user/:repo/deployments
#### GET
Fetches a list of previous deployments for which happened for the related git repo eg. `github.com/zetaron/deployment-bot-thing`

#### Discussion:
- Do we care for users who are users of eg. GitLab but are self-hosting and therefore do own a custom domain? or do we suppose that they are able to setup thier own Provider API to facilitate this fact?

### /github/:user/:repo/deployment/:id
#### GET
Has all the features as `/deployment/:id` with the additional benefit of Authentication, Scoped access, access to the original request made to the Provider API and the resulting mapped request which got sent to the API.

```json
{
  "original_request": {}, // what was received by the Provider API
  "original_mapped_request": {}, // what was sent to the API
}
```

### /github/:user/:repo/deployment/:id/logs
Has all the features as `/deployment/:id/logs` with the only benefit of Authentication and Scoped access.
