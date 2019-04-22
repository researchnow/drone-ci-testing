# drone-ci-testing [![Build Status](https://drone.rbkr.xyz/api/badges/ruanbekker/drone-ci-testing/status.svg)](https://drone.rbkr.xyz/ruanbekker/drone-ci-testing)
testing cloud.drone.io

## Contents

Contents of this page

* [Setups]()
  * [Drone CLI Setup(#drone-cli-setup)

* [Pipeline Examples]()
  * [Shell Commands in a Step](#run-shell-commands-in-a-step)
  * [Secrets in Environment Variables](#secrets-in-environment-variables)
  * [Publish to Dockerhub](#publish-to-dockerhub)
  * [Publish to S3](#publish-to-s3-object-storage)
  * [Slack Notifications](#slack-notifications)
  * [Telegram Notifications](#telegram-notifications)
  * [SSH Example](#ssh-example)
  * [SCP Example](#scp-example)
  * [Full Pipeline Examples](#pipeline-examples)

## Drone CLI Setup

[Docs](https://docs.drone.io/cli/install/)

```
curl -L https://github.com/drone/drone-cli/releases/download/v1.0.7/drone_darwin_amd64.tar.gz | tar zx
sudo cp drone /usr/local/bin
```

Get your account details at https://drone.your-domain.com/account

```
export DRONE_SERVER=https://yourdomain
export DRONE_TOKEN=xx
drone info
```

## Usage

### Run shell commands in a step

```
kind: pipeline
name: default

steps:
  - name: test
    image: alpine
    environment:
      MYUSERNAME:
        from_secret: myusername
    commands:
      - echo "echo secret"
      - echo $${MYUSERNAME}
      - echo "run env command"
      - env
```

or:

```
pipeline:
  docker:
    image: alpine
    secrets: [ myusername ]
    commands:
      - echo "echo secret"
      - echo $${MYUSERNAME}
      - echo "run env command"
      - env
```

This will output:

```
+ echo "echo secret"
echo secret
+ echo ${MYUSERNAME}
********
+ echo "run env command"
run env command
+ env
DRONE_SYSTEM_HOST=cloud.drone.io
DRONE_COMMIT_AUTHOR_AVATAR=https://avatars0.githubusercontent.com/u/567298?v=4
DRONE_BRANCH=master
DRONE_GIT_SSH_URL=git@github.com:ruanbekker/drone-ci-testing.git
DRONE_JOB_FINISHED=1555573281
CI=true
HOSTNAME=7a20b000f25c
CI_BUILD_STARTED=1555573280
DRONE_REPO_LINK=https://github.com/ruanbekker/drone-ci-testing
DRONE_TARGET_BRANCH=master
DRONE_REPO_NAMESPACE=ruanbekker
DRONE_STAGE_OS=linux
DRONE_COMMIT_AUTHOR=ruanbekker
CI_WORKSPACE=/drone/src
DRONE_GIT_HTTP_URL=https://github.com/ruanbekker/drone-ci-testing.git
DRONE_RUNNER_HOSTNAME=15e89c0f84f1
SHLVL=1
DRONE_COMMIT_BRANCH=master
HOME=/root
DRONE_REPO_SCM=
DRONE_REPO_PRIVATE=false
DRONE_SYSTEM_PROTO=https
DRONE_STEP_NUMBER=2
DRONE_REPO_VISIBILITY=public
DRONE_BUILD_STATUS=success
DRONE_BUILD_ACTION=
DRONE_RUNNER_PLATFORM=linux/amd64
DRONE_WORKSPACE_BASE=/drone/src
DRONE_COMMIT_BEFORE=138bc48338f44be2ef4ad8a57d63d4bf2403da3c
CI_JOB_STARTED=1555573280
DRONE_STAGE_ARCH=amd64
CI_WORKSPACE_PATH=
DRONE_STAGE_STATUS=success
DRONE_SOURCE_BRANCH=master
DRONE_STAGE_NAME=default
DRONE_REPO_BRANCH=master
DRONE_COMMIT_MESSAGE=add envvar
DRONE_DEPLOY_TO=
CI_BUILD_FINISHED=1555573281
DRONE_SYSTEM_HOSTNAME=cloud.drone.io
DRONE_REMOTE_URL=https://github.com/ruanbekker/drone-ci-testing.git
DRONE_JOB_STATUS=success
MYUSERNAME=********
DRONE_REPO_OWNER=ruanbekker
DRONE_STAGE_KIND=pipeline
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
DRONE=true
DRONE_BUILD_NUMBER=15
DRONE_BUILD_STARTED=1555573280
DRONE_BUILD_LINK=https://cloud.drone.io/ruanbekker/drone-ci-testing/15
CI_JOB_FINISHED=1555573281
DRONE_WORKSPACE=/drone/src
DRONE_STAGE_VARIANT=
DRONE_STAGE_DEPENDS_ON=
DRONE_COMMIT_AFTER=05818a52403e07b2cab4d980522e7264be4aa18c
DRONE_STAGE_NUMBER=1
DRONE_STAGE_STARTED=1555573280
DRONE_COMMIT=05818a52403e07b2cab4d980522e7264be4aa18c
DRONE_RUNNER_HOST=15e89c0f84f1
DRONE_JOB_STARTED=1555573280
DRONE_WORKSPACE_PATH=
DRONE_BUILD_EVENT=push
DRONE_COMMIT_SHA=05818a52403e07b2cab4d980522e7264be4aa18c
DRONE_BUILD_CREATED=1555573279
DRONE_COMMIT_AUTHOR_EMAIL=ruan.ru.bekker@gmail.com
DRONE_REPO_NAME=drone-ci-testing
DRONE_STAGE_MACHINE=15e89c0f84f1
DRONE_COMMIT_REF=refs/heads/master
CI_BUILD_STATUS=success
DRONE_MACHINE=15e89c0f84f1
CI_WORKSPACE_BASE=/drone/src
PWD=/drone/src
DRONE_BUILD_FINISHED=1555573281
DRONE_COMMIT_LINK=https://github.com/ruanbekker/drone-ci-testing/compare/138bc48338f4...05818a52403e
DRONE_REPO=ruanbekker/drone-ci-testing
DRONE_STEP_NAME=test
DRONE_SYSTEM_VERSION=1.0.0
DRONE_COMMIT_AUTHOR_NAME=Ruan Bekker
CI_JOB_STATUS=success
DRONE_STAGE_FINISHED=1555573281
```

### Secrets in Environment Variables

Secrets are exposed to your pipeline steps and can be referenced with uppercase names: [reference](https://discourse.drone.io/t/secret-problems/2863)

```
pipeline:
  docker:
    image: docker
    secrets: [ swarm_key ]
    commands:
    - echo $${SWARM_KEY}
    when:
      branch: [master, develop, release/*]
```

will result in:

```
echo ${SWARM_KEY}
-----rsa key ----- 
```

### Publish to Dockerhub:
- http://plugins.drone.io/drone-plugins/drone-docker/

- Clone
- Build Docker Image
- Publish to Dockerhub with `latest` and `git_commit_id` and use credentials from stored secrets

The `.drone.yml`

```
kind: pipeline
name: default

steps:
  - name: test
    image: alpine
    environment:
      MYUSERNAME:
        from_secret: myusername
    commands:
      - echo "echo secret"
      - echo $${MYUSERNAME}
      - echo "run env command"
      - env
      - echo $${MYUSERNAME} > /tmp/file
      - mkdir -p /tmp/foo/dir
      - touch /tmp/foo/dir/test.txt
      
  - name: publish
    image: plugins/docker
    settings:
      repo: ruanbekker/dronetest
      auto_tag: false
      auto_tag_suffix: alpine
      tags:
        - ${DRONE_COMMIT}
        - latest
      username: 
        from_secret: dockerhub_username
      password:
        from_secret: dockerhub_password
```

### Publish to S3 Object Storage

Publish data under `public/*` to minio object stroage:
- http://plugins.drone.io/drone-plugins/drone-s3/

```
kind: pipeline
name: default

steps:
  - name: test
    ..
      
  - name: publish
    ..
        
  - name: upload
    image: plugins/s3
    settings:
      bucket: dump
      source: public/*
      target: /path/to
      path_style: true
      access_key:
        from_secret: minio_access_key
      secret_key:
        from_secret: minio_secret_key
      endpoint: 
        from_secret: minio_host

```

Verifyin the output:

```
❯ aws --profile objects s3 --endpoint-url=https://objects.domain.com ls --recursive s3://dump/
2019-04-18 10:47:30          4 path/to/public/data.txt
```

### Slack Notifications:

- http://plugins.drone.io/drone-plugins/drone-slack/

```
  - name: slack
    image: plugins/slack
    settings:
      webhook:
        from_secret: slack_webhook
      channel: system_events
      image_url: https://unsplash.it/256/256/?random
      icon_url: https://unsplash.it/256/256/?random
      template: >
        {{#success build.status}}
          build {{build.number}} succeeded. Good job. link: {{commit.link}} .
        {{else}}
          build {{build.number}} failed. Fix me please.
        {{/success}}
```

More verbose info:

```
kind: pipeline
name: default

steps:
..

  - name: slack
    image: plugins/slack
    settings:
      webhook:
        from_secret: slack_webhook
      channel: system_events
      icon_url: https://www.pngkit.com/png/full/92-920674_drone-logo-png-transparent-drone-ci-logo.png
      template: >
        {{#success build.status}}
           build {{build.number}} status: *{{build.status}}*
          build link: {{build.link}}
          build author: {{build.author}}
          build ref: {{build.ref}}
          build event: {{build.event}}
          build commit: {{build.commit}}
          build branch: {{build.branch}}
          build deploy: {{build.deployTo}}
        {{else}}
          build {{build.number}} failed. Fix me please.
        {{/success}}

```

### Telegram Notifications:

```
..
- name: telegram
  image: appleboy/drone-telegram
  settings:
    token:
      from_secret: telegram_token
    to:
      from_secret: telegram_to
  when:
    status: [ success, failure ]
  photo:
    - https://cdn.shopify.com/s/files/1/1061/1924/products/Virus_Emoji_large.png?v=1480481048
  format: markdown
```

### SSH Example

- http://plugins.drone.io/appleboy/drone-ssh/

```
...
- name: ssh
  image: appleboy/drone-ssh
  pull: true
  settings:
    username: root
    port: 2222
    host:
      from_secret: ssh_host
    key:
        from_secret: ssh_key
    script:
      - ls -lah
      - find / -name Dockerfile
      - docker node ls
```

### SCP Example

- https://github.com/appleboy/drone-scp

```
 myscp-step:
   image: appleboy/drone-scp
   host: 
     from_secret: swarm_host
   username:
     from_secret: swarm_user
   key:
     from_secret: swarm_key
   target: /root/context
   source: /root
   when:
     event: [push]
```


### Example Pipelines

- [myth/overflow drone pipeline](https://github.com/myth/overflow/blob/master/.drone.yml)
- [drone/docs pipeline](https://github.com/drone/docs/blob/master/.drone.yml)

### Docker Swarm Stacks

- https://github.com/codestation/drone-stack

```

```
