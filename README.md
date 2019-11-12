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

#### 2.1 手动添加

`"ADMIN"` -> `"Pipelines"` -> `"+ Create a new pipeline ..."`

**Conditional task execution**

You can use the run_if attribute of the task to achieve that. It takes passed, failed or any. Passed is the default value, that is only if the current task is passed it moves on to the next one. Failed is the exact opposite of this. I generally use it for setting up assertions within the job. Any is basically you don't care the outcome of that task. GoCD will not fail irrespective of the outcome of that task within the job.

#### 2.2 pipeline as code
click `"ADMIN"` -> `"Config XML"` -> `"EDIT"` ,then add  below context
```bash
<?xml version="1.0" encoding="utf-8"?>
<cruise xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="cruise-config.xsd" schemaVersion="132">
  <server agentAutoRegisterKey="e5f55b50-3a5a-4885-8662-f1855b0817fb" webhookSecret="06d80a08-c06a-4bea-b6de-0859f275a33a" commandRepositoryLocation="default" serverId="99983723-9b58-4790-a395-bdf2ab87981f" tokenGenerationKey="ba6d146b-5265-4a83-950d-70cc069fb7a0">
    <backup emailOnSuccess="true" emailOnFailure="true" />
    <artifacts>
      <artifactsDir>artifacts</artifactsDir>
    </artifacts>
  </server>

<!-- New added,begin(新加的) -->
  <config-repos>
    <config-repo pluginId="yaml.config.plugin" id="gocd-node-demo">
      <git url="https://github.com/cicd-draft/node-demo.git" />
    </config-repo>

    <config-repo pluginId="yaml.config.plugin" id="gocd-node-demo2">
      <git url="https://github.com/cicd-draft/node-demo.git" />
    </config-repo>
    ...

  </config-repos>
<!-- New added,end(新加的) -->

  <pipelines group="defaultGroup" />
</cruise>
```

### 3. 插件
[GoCD PLUGINS](https://www.gocd.org/plugins)
> 有些竟然是需要付费的 >::<
- script executor task
每个task只能执行一条命令，导致每个job都有十几个task这个插件可以让你像shell脚本一样编排指令，从而愉快地合并臃肿的task

- docker task

--- 
More ref:

- [set up doc](https://janikvonrotz.ch/2018/11/06/setup-gocd-environment-using-docker/)
- [gocd doc](https://docs.gocd.org/current/introduction/concepts_in_go.html)
- [GoCD Configuration Reference](https://docs.gocd.org/current/configuration/configuration_reference.html#cruise)
- [gocd-yaml-config-plugin](https://github.com/tomzo/gocd-yaml-config-plugin)