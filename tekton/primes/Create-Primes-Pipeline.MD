# Primes Tekton Pipeline

## Introduction
After some experimentation with openshift and tekton I decided that it would be good to get a decent pipeline nailed in 
vanilla tekton and then look to move that to OCP. The basic outcome I am looking to achieve is:

```text
Git clone --> Gradle clean build test --> Sonarqube scan --> Docker build --> Docker push 
```

## Pre-requisites

Before starting install:

- Tekton on your kubernetes cluster 
- `tkn` cli
- set the persistent volume details to be used by tekton

These are described [here](https://tekton.dev/docs/getting-started/)

## Building the pipeline

### Task 1 - git clone

On Tekton hub there is a git clone task already defined [here](https://github.com/tektoncd/catalog/blob/main/task/git-clone/0.4/git-clone.yaml) 
and so I decided to utilise this (saving it [here](./tasks/tekton-git-clone.yaml)). I checked it worked by defining a 
`TaskRun` see [here](./taskruns/test-taskrun-git-clone.yaml)

This worked and so I added to the [primes pipeline](./pipelines/primes-pipeline.yaml) and ran it with a pipeline run 
which gratifyingly gave successful output:

```
[clone-repository : clone] + '[' false '=' true ]
[clone-repository : clone] + '[' false '=' true ]
[clone-repository : clone] + CHECKOUT_DIR=/workspace/output/
[clone-repository : clone] + '[' true '=' true ]
[clone-repository : clone] + cleandir
[clone-repository : clone] + '[' -d /workspace/output/ ]
[clone-repository : clone] + rm -rf '/workspace/output//*'
[clone-repository : clone] + rm -rf '/workspace/output//.[!.]*'
[clone-repository : clone] + rm -rf '/workspace/output//..?*'
[clone-repository : clone] + test -z 
[clone-repository : clone] + test -z 
[clone-repository : clone] + test -z 
[clone-repository : clone] + /ko-app/git-init '-url=https://github.com/edoatley/primes.git' '-revision=master' '-refspec=' '-path=/workspace/output/' '-sslVerify=true' '-submodules=true' '-depth=1' '-sparseCheckoutDirectories='
[clone-repository : clone] {"level":"info","ts":1632661374.2056837,"caller":"git/git.go:169","msg":"Successfully cloned https://github.com/edoatley/primes.git @ 70760d0b482266d56da27f1123ed176af13d2e50 (grafted, HEAD, origin/master) in path /workspace/output/"}
[clone-repository : clone] {"level":"info","ts":1632661374.223901,"caller":"git/git.go:207","msg":"Successfully initialized and updated submodules in path /workspace/output/"}
[clone-repository : clone] + cd /workspace/output/
[clone-repository : clone] + git rev-parse HEAD
[clone-repository : clone] + RESULT_SHA=70760d0b482266d56da27f1123ed176af13d2e50
[clone-repository : clone] + EXIT_CODE=0
[clone-repository : clone] + '[' 0 '!=' 0 ]
[clone-repository : clone] + printf '%s' 70760d0b482266d56da27f1123ed176af13d2e50
[clone-repository : clone] + printf '%s' https://github.com/edoatley/primes.git
```

### Task 2 - gradle build

