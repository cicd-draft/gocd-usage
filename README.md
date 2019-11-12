# gocd-usage
How to setup gocd, then introduce some basic usage



### 1. Setup GoCD environment using docker
Pull official GoCD server and agent images   
```bash
docker pull gocd-server:v19.10.0
docker pull gocd/gocd-agent-alpine-3.9:v19.10.0
```

#### 1.1 Start a new GoCD server container
`docker run -d --name gocd-server -p 8153:8153 -p 8154:8154 gocd/gocd-server:v19.10.0`

In your browser open https://localhost:8154.

#### 1.2 Start a new GoCD agent container

```bash
docker run -itd --name gocd-agent \
-e CI=true \
-e GO_SERVER_URL=https://$(docker inspect --format='{{(index (index .NetworkSettings.IPAddress))}}' gocd-server):8154/go \
gocd/gocd-agent-alpine-3.9:v19.10.0
 ```

The agent is registering itself using the GO_SERVER_URL variable and should be listed in the management interface https://localhost:8154/go/agents.
If the GoCD agent entered the state pending, select it and click on Enable.

>Or you can use **docker-compose.yml**(which is located the root dir) to setup the environment.

#### 1.3 Agent configuration

These binaries are not installed on the image by default, so we need to update the image.

Log into the gocd-agent container.
`docker exec -it -u root gocd-agent bash`

example:
`apk add --no-cache nodejs yarn`


### 2. Pipeline example 

https://github.com/cicd-draft/node-demo


--- 
More ref:

- [set up doc](https://janikvonrotz.ch/2018/11/06/setup-gocd-environment-using-docker/)
- [gocd doc](https://docs.gocd.org/current/introduction/concepts_in_go.html)