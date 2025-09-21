# Concourse

## Setup

```bash
curl -O https://concourse-ci.org/docker-compose.yml
docker-compose up -d

#  ✔ Network concourse_default           Created                 0.0s
#  ✔ Container concourse-concourse-db-1  Healthy                 4.6s
#  ✔ Container concourse-concourse-1     Started                 4.2s

# Concourse will be running at localhost:8080 on your machine. You can log in with the username/password as test/test.

# check docker containers are running related to concourse
docker ps

# CONTAINER ID   IMAGE                 COMMAND                  CREATED        STATUS                  PORTS                                       NAMES
# cf0bb901e5ee   concourse/concourse   "dumb-init /usr/loca…"   14 hours ago   Up 14 hours             0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   concourse-concourse-1
# d8e75f495607   postgres              "docker-entrypoint.s…"   14 hours ago   Up 14 hours (healthy)   5432/tcp                                    concourse-concourse-db-1

# TODO: Deploy to k8s local using minikube via helm charts

# Download CLI Client 

curl 'http://localhost:8080/api/v1/cli?arch=arm64&platform=darwin' -o fly
chmod +x ./fly
mv ./fly /usr/local/bin/

fly -t tutorial login -c http://localhost:8080 -u test -p test
# logging in to team 'main'

# target saved

# TODO: Use it with OIDC

# make a file `hello-world.yml` with following content
jobs:
- name: hello-world-job
  plan:
  - task: hello-world-task
    config:
      # Tells Concourse which type of worker this task should run on
      platform: linux
      # This is one way of telling Concourse which container image to use for a
      # task. We'll explain this more when talking about resources
      image_resource:
        type: registry-image
        source:
          repository: busybox # images are pulled from docker hub by default
          tag: latest
      # The command Concourse will run inside the container
      # echo "Hello world!"
      run:
        path: echo
        args: ["Hello world!"]


fly -t tutorial set-pipeline -p hello-world -c pipelines/hello-world.yml
# jobs:
#   job hello-world-job has been added:
# + name: hello-world-job
# + plan:
# + - config:
# +     image_resource:
# +       name: ""
# +       source:
# +         repository: busybox
# +         tag: latest
# +       type: registry-image
# +     platform: linux
# +     run:
# +       args:
# +       - Hello world!
# +       path: echo
# +   task: hello-world-task

# pipeline name: hello-world

# apply configuration? [yN]: y
# pipeline created!
# you can view your pipeline here: http://localhost:8080/teams/main/pipelines/hello-world

# the pipeline is currently paused. to unpause, either:
#   - run the unpause-pipeline command:
#     fly -t tutorial unpause-pipeline -p hello-world
#   - click play next to the pipeline in the web ui

# pipelines are paused when first created
fly -t tutorial unpause-pipeline -p hello-world
# unpaused 'hello-world'

# trigger the job and watch it run to completion
fly -t tutorial trigger-job --job hello-world/hello-world-job --watch

# started hello-world/hello-world-job #1

# initializing
# initializing check: image
# selected worker: cf0bb901e5ee
# selected worker: cf0bb901e5ee
# fetching busybox@sha256:d82f458899c9696cb26a7c02d5568f81c8c8223f8661bb2a7988b269c8b9051e
# 499bcf3c8ead [============================================] 1.8MiB/1.8MiB
# selected worker: cf0bb901e5ee
# running echo Hello world!
# Hello world!
# succeeded

---

# Let us set up another pipeline task called read the README.md (read-the-readme)

resources:
- name: concourse-examples
  type: git
  icon: github
  source:
    uri: https://github.com/concourse/examples

jobs:
- name: read-the-readme
  plan:
  - get: concourse-examples
  - task: cat-readme
    config:
      platform: linux
      image_resource:
        type: registry-image
        source:
          repository: busybox
      inputs: # pass concourse-examples into this task step
      - name: concourse-examples
      run:
        path: cat
        args: ["concourse-examples/README.md"]

# run the pipeline
fly -t tutorial set-pipeline -p read-the-readme -c pipelines/read-the-readme.yml

# resources:
#   resource concourse-examples has been added:
# + icon: github
# + name: concourse-examples
# + source:
# +   uri: https://github.com/concourse/examples
# + type: git

# jobs:
#   job read-the-readme has been added:
# + name: read-the-readme
# + plan:
# + - get: concourse-examples
# + - config:
# +     image_resource:
# +       name: ""
# +       source:
# +         repository: busybox
# +       type: registry-image
# +     inputs:
# +     - name: concourse-examples
# +     platform: linux
# +     run:
# +       args:
# +       - concourse-examples/README.md
# +       path: cat
# +   task: cat-readme

# pipeline name: read-the-readme

# apply configuration? [yN]: y
# pipeline created!
# you can view your pipeline here: http://localhost:8080/teams/main/pipelines/read-the-readme

# the pipeline is currently paused. to unpause, either:
#   - run the unpause-pipeline command:
#     fly -t tutorial unpause-pipeline -p read-the-readme
#   - click play next to the pipeline in the web ui

fly -t tutorial unpause-pipeline -p read-the-readme
# unpaused 'read-the-readme'

fly -t tutorial trigger-job --job read-the-readme/read-the-readme --watch

# started read-the-readme/read-the-readme #1

# selected worker: cf0bb901e5ee
# Cloning into '/tmp/build/get'...
# fbd930e bump image tag
# initializing
# initializing check: image
# selected worker: cf0bb901e5ee
# selected worker: cf0bb901e5ee
# fetching busybox@sha256:d82f458899c9696cb26a7c02d5568f81c8c8223f8661bb2a7988b269c8b9051e
# 499bcf3c8ead [============================================] 1.8MiB/1.8MiB
# selected worker: cf0bb901e5ee
# running cat concourse-examples/README.md
# # examples
# Examples of Concourse workflows
# succeeded

fly -t tutorial trigger-job --job read-the-readme/read-the-readme --watch
# started read-the-readme/read-the-readme #2

# selected worker: cf0bb901e5ee
# INFO: found existing resource cache

# initializing
# initializing check: image
# selected worker: cf0bb901e5ee
# selected worker: cf0bb901e5ee
# INFO: found existing resource cache

# selected worker: cf0bb901e5ee
# running cat concourse-examples/README.md
# # examples
# Examples of Concourse workflows
# succeeded
```

### Small Go Project

When it comes to real-world projects, task configs are usually stored alongside the code that it's testing. This makes the pipeline a bit smaller and makes the config able to change without needing to reconfigure the pipeline.

```bash
# create a file with multiline content using cat
cat <<EOF > pipelines/small-go-project-test.yml
---
resources:
- name: booklit
  type: git
  source:
    uri: https://github.com/concourse/booklit
    branch: master

jobs:
- name: unit
  plan:
  - get: booklit
    trigger: true
  - task: unit
    file: booklit/ci/test.yml
EOF 

# Set Pipeline
fly -t tutorial set-pipeline -p small-go-project-test -c pipelines/small-go-project-test.yml 
# resources:
#   resource booklit has been added:
# + name: booklit
# + source:
# +   branch: master
# +   uri: https://github.com/concourse/booklit
# + type: git
  
# jobs:
#   job unit has been added:
# + name: unit
# + plan:
# + - get: booklit
# +   trigger: true
# + - file: booklit/ci/test.yml
# +   task: unit
  
# pipeline name: small-go-project-test

# apply configuration? [yN]: y
# pipeline created!
# you can view your pipeline here: http://localhost:8080/teams/main/pipelines/small-go-project-test

# the pipeline is currently paused. to unpause, either:
#   - run the unpause-pipeline command:
#     fly -t tutorial unpause-pipeline -p small-go-project-test
#   - click play next to the pipeline in the web ui

# Unpause the pipeline
fly -t tutorial unpause-pipeline -p small-go-project-test
# unpaused 'small-go-project-test'

# Trigger the Job
fly -t tutorial trigger-job --job small-go-project-test/unit --watch
# started small-go-project-test/unit #2

# selected worker: cf0bb901e5ee
# INFO: found existing resource cache

# initializing
# initializing check: image
# selected worker: cf0bb901e5ee
# selected worker: cf0bb901e5ee
# INFO: found existing resource cache

# selected worker: cf0bb901e5ee
# running booklit/ci/test
# installing ginkgo...
# running tests...
# warning: no packages being tested depend on matches for pattern github.com/vito/booklit/booklitcmd
# warning: no packages being tested depend on matches for pattern github.com/vito/booklit/chroma
# warning: no packages being tested depend on matches for pattern github.com/vito/booklit/chroma/plugin
# warning: no packages being tested depend on matches for pattern github.com/vito/booklit/docs/go

# Running Suite: Booklit Suite
# ============================
# Random Seed: 1758378207
# Will run 62 specs

# Running in parallel across 7 nodes

# ••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••
# Ran 62 of 62 Specs in 0.131 seconds
# SUCCESS! -- 62 Passed | 0 Failed | 0 Pending | 0 Skipped


# Ginkgo ran 1 suite in 3.029009418s
# Test Suite Passed
# succeeded

```

### Chaining Jobs

Promoting resources to downstream jobs is done by setting get step passed on a get step.

Note that nothing in unit says anything about triggering build. Job definitions are self-contained; they describe their dependencies and where they come from, which results in a dependency flow that Concourse pushes forward.

```bash
cat << EOF > pipelines/small-go-project-test-build.yml
---
resources:
- name: booklit
  type: git
  source:
    uri: https://github.com/concourse/booklit
    branch: master

jobs:
- name: unit
  plan:
  - get: booklit
    trigger: true
  - task: run-unit
    file: booklit/ci/test.yml

- name: build
  plan:
  - get: booklit
    passed: [unit]
    trigger: true
  - task: run-build
    file: booklit/ci/build.yml
EOF

# Set pipeline with the file we created

fly -t tutorial set-pipeline -p small-go-project-test-build -c pipelines/small-go-project-test-build.yml
# resources:
#   resource booklit has been added:
# + name: booklit
# + source:
# +   branch: master
# +   uri: https://github.com/concourse/booklit
# + type: git
  
# jobs:
#   job unit has been added:
# + name: unit
# + plan:
# + - get: booklit
# +   trigger: true
# + - file: booklit/ci/test.yml
# +   task: run-unit
  
#   job build has been added:
# + name: build
# + plan:
# + - get: booklit
# +   passed:
# +   - unit
# +   trigger: true
# + - file: booklit/ci/build.yml
# +   task: run-build
  
# pipeline name: small-go-project-test-build

# apply configuration? [yN]: y
# pipeline created!
# you can view your pipeline here: http://localhost:8080/teams/main/pipelines/small-go-project-test-build

# the pipeline is currently paused. to unpause, either:
#   - run the unpause-pipeline command:
#     fly -t tutorial unpause-pipeline -p small-go-project-test-build
#   - click play next to the pipeline in the web ui


# Unpause the pipeline
fly -t tutorial unpause-pipeline -p small-go-project-test-build
# unpaused 'small-go-project-test-build'

fly -t tutorial trigger-job --job small-go-project-test-build/build --watch
# started small-go-project-test-build/build #1

# selected worker: cf0bb901e5ee
# INFO: found existing resource cache

# initializing
# initializing check: image
# selected worker: cf0bb901e5ee
# selected worker: cf0bb901e5ee
# INFO: found existing resource cache

# selected worker: cf0bb901e5ee
# running booklit/ci/build
# /tmp/build/4afbfa8e/booklit /tmp/build/4afbfa8e
# building linux binary...
# go: downloading github.com/sirupsen/logrus v1.7.0
# go: downloading github.com/jessevdk/go-flags v1.4.0
# go: downloading github.com/agext/levenshtein v1.2.3
# go: downloading github.com/segmentio/textio v1.2.0
# go: downloading golang.org/x/sys v0.0.0-20201218084310-7d0127a74742
# building darwin binary...
# building windows binary...
# # golang.org/x/sys/windows
# ../gopath/pkg/mod/golang.org/x/sys@v0.0.0-20201218084310-7d0127a74742/windows/types_windows.go:1597:24: undefined: JOBOBJECT_BASIC_LIMIT_INFORMATION
# ../gopath/pkg/mod/golang.org/x/sys@v0.0.0-20201218084310-7d0127a74742/windows/zsyscall_windows.go:2989:38: undefined: WSAData
# ../gopath/pkg/mod/golang.org/x/sys@v0.0.0-20201218084310-7d0127a74742/windows/zsyscall_windows.go:3065:51: undefined: Servent
# ../gopath/pkg/mod/golang.org/x/sys@v0.0.0-20201218084310-7d0127a74742/windows/zsyscall_windows.go:3079:50: undefined: Servent
# ../gopath/pkg/mod/golang.org/x/sys@v0.0.0-20201218084310-7d0127a74742/windows/zsyscall_windows.go:3081:8: undefined: Servent
# failed

```

### Using Resource Types

Resource Types can be used to extend the functionality of your pipeline and provide deeper integrations. This example uses one to trigger a job whenever a new Dinosaur Comic is out.

```bash
cat << EOF > pipelines/comics-rss-announce.yml
---
resource_types:
- name: rss
  type: registry-image
  source:
    repository: suhlig/concourse-rss-resource
    tag: latest

resources:
- name: dinosaur-comics
  type: rss
  source:
    url: http://www.qwantz.com/rssfeed.php

jobs:
- name: announce
  plan:
  - get: dinosaur-comics
    trigger: true
EOF

# 

```


## Operations

```bash
# List Workers
# fly -t tutorial workers
fly -t tutorial ws
name          containers  platform  tags  team  state    version  age
cf0bb901e5ee  1           linux     none  none  running  2.5      9h36m


# List Volumes
# fly -t tutorial volumes
fly -t tutorial vs
# handle                                worker        type           identifier
# 127d2fe8-3e87-432e-b889-0cb1376f5f7f  cf0bb901e5ee  resource-type  git
# 1ff6c1eb-07f9-49e4-a7a0-9221d367ec81  cf0bb901e5ee  container      5f4d3238-f72e-4d8d-b4c3-acef18601f68
# 29e08e7f-5686-44d4-bd42-0d163e3e4ada  cf0bb901e5ee  container      2c043aca-1536-4a14-8fec-703624fdaf4c
# 333b24d2-0f6c-441e-87cd-322b1a216d2c  cf0bb901e5ee  resource-type  registry-image
# 4989cf33-fe2d-4b83-bcf2-34b35e1e498d  cf0bb901e5ee  resource       ref:fbd930eebc8323575ab282c764ef0f6620898d07
# 906616c3-e789-4384-8a53-d890962c6ed8  cf0bb901e5ee  container      2c043aca-1536-4a14-8fec-703624fdaf4c
# 93b49093-4559-42c1-a747-0b89ddab6128  cf0bb901e5ee  container      7c5338b9-e804-46d2-a8e9-a68695d7a8f5
# 97453b9c-b457-41a9-a143-535984a35a34  cf0bb901e5ee  container      7c5338b9-e804-46d2-a8e9-a68695d7a8f5
# 9c4dffd6-cd26-46b1-99e4-aa0c5635dd6e  cf0bb901e5ee  container      2c043aca-1536-4a14-8fec-703624fdaf4c
# a3f73305-6838-4e3a-89c1-744a3a9ba91d  cf0bb901e5ee  resource       digest:sha256:d82f458899c9696cb26a7c02d5568f81c8c8223f8661bb2a7988b269c8b9051e,tag:1.37.0
# b2db94f9-e0da-4364-a849-80467f6c3b0e  cf0bb901e5ee  resource       digest:sha256:d82f458899c9696cb26a7c02d5568f81c8c8223f8661bb2a7988b269c8b9051e,tag:latest
# d73b2f14-4423-41f8-87ea-26854c219e4e  cf0bb901e5ee  container      5f4d3238-f72e-4d8d-b4c3-acef18601f68
# f0742ba4-95f9-4d23-9325-979c001d0236  cf0bb901e5ee  container      2c043aca-1536-4a14-8fec-703624fdaf4c


# List pipelines
fly -t tutorial pipelines
# id  name             paused  public  last updated
# 1   hello-world      no      no      2025-09-20 01:47:34 +0200 CEST
# 2   read-the-readme  no      no      2025-09-20 02:02:47 +0200 CEST


```
