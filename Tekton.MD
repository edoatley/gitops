Tekton Investigation
===================

## Set up

1. Set up a Persistent Volume - applied [pv.yaml](tekton/experimenting/pv.yaml)
> To run a CI/CD workflow, you need to provide Tekton a Persistent Volume 
> for storage purposes. Tekton requests a volume of 5Gi with the default storage class by default.
2. Install `tkn` CLI:
```shell
brew tap tektoncd/tools
brew install tektoncd/tools/tektoncd-cli
```
## First Task

1. Create a simple hello world [task](tekton/experimenting/first-task.yaml)
2. Dry run a TaskRun with `tkn`:
```shell
$ tkn task start hello --dry-run
tkn task start hello --dry-run
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  creationTimestamp: null
  generateName: hello-run-
  namespace: default
spec:
  resources: {}
  serviceAccountName: ""
  taskRef:
    name: hello
status:
  podName: ""
```
3. Run the task for real:
```shell
$ tkn task start hello --dry-run                                                 Py base 20:21:09
          
TaskRun started: hello-run-w5wbz

In order to track the TaskRun progress run:
tkn taskrun logs hello-run-w5wbz -f -n default
```
4. Review Logs from the task run:
```shell
$ tkn taskrun logs hello-run-w5wbz -f -n default                                 Py base 20:21:51
tkn taskrun logs hello-run-w5wbz -f -n default
[hello] Hello World!
```