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

The gradle build task is fairly simple - it shares the workspace created to checkout the repository and uses the gradle 
image to allow the project to be built using gradle. As I was refactoring the task to ensure it was working correctly I 
was executing the following debug cycle:

```shell
$ k delete $(kubectl get pods --field-selector=status.phase!=Running -o name)
pod "primes-build-run-cvljj-build-and-test-qrv66-pod-cpcls" deleted
pod "primes-build-run-cvljj-clone-repository-6dxjv-pod-shp8d" deleted

$ k apply -f tekton/primes/tasks/gradle.yaml
task.tekton.dev/gradle configured

$ k create -f tekton/primes/pipelineruns/run-primes-pipeline.yaml                                                                        Py base ○ docker-desktop/tekton 19:17:54
pipelinerun.tekton.dev/primes-build-run-tdzj8 created

$ tkn pr logs primes-build-run-tdzj8
#<log output>
```

The end state `Task` essentially just takes a workspace with the source and applies a executes a provided list of tasks
with a given set of arguments.

### Task 3 - sonarqube (commented out for now)

This task is simply reusing the build task from step 2 to execute the sonarqube task with gradle. It will need some
credentials provided by a secret.

### Task 4 - mutation test

This task is simply reusing the build task from step 2 to execute the pitest task with gradle.

### Task 5 - buildah

This task is responsible for building the docker image - luckily buildah has a tekton task which is defined [here](https://github.com/tektoncd/catalog/blob/main/task/buildah/0.2/buildah.yaml)
and saved to [tekton-buildah.yaml](./tasks/tekton-buildah.yaml)

One bit of set up is the authentication to docker which I did as follows:

```shell
kubectl create secret -n tekton docker-registry container-registry-secret \
--docker-server=https://index.docker.io/v2/ \
--docker-username=edoatley \
--docker-password=<password>

kubectl create sa -n tekton build-bot

kubectl patch serviceaccount build-bot \
-p '{"secrets": [{"name": "container-registry-secret"}]}'
```

Then use the `build-bot` service account in the pipeline run

