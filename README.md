# gitops
PLace to play with RH Gitops in openshift

## References

- This repository is inspired by [Dev Epihany Blog](https://devepiphany.org/2021/04/27/geitting-to-grips-with-gitops/)
- This is useful info [Getting Started with OpenShift GitOps](https://github.com/siamaksade/openshift-gitops-getting-started)
- [Official Pipelines Information](https://cloud.redhat.com/learn/topics/ci-cd)

## Prerequisites

### Create an Openshift Instance

* Create a project on GCP
* Download IPI and pull secret from RedHat
* Follow documentation and this [great article](https://cloud.redhat.com/blog/ocp-4.6-install-on-gcp-cloud-the-smooth-experience)
* Lots of trial and error when GCP misbehaved with errors I did not unterstand as never used it before!

## Create Identity provider

1. Went with a simple htpasswd 

```shell
htpasswd -c .htpasswd user
New password: 
Re-type new password: 
Updating password for user user
~/code/openshift/htpasswd ❯ htpasswd .htpasswd user2                                                                                                                                                        4s Py base 09:29:36
New password: 
Re-type new password: 
Adding password for user user2
```

then applied:

```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
    - name: htpasswd-provider
      mappingMethod: claim
      type: HTPasswd
      htpasswd:
        fileData:
          name: htpass-secret

```

2. Logged on to UI as both
3. Created RoleBindings

## Following along the blog

1. Login as Admin and run

```shell
oc extract secret/openshift-gitops-cluster -n openshift-gitops --to=-                                              
# admin.password
<this is the password value for user admin>
```

2. From the [Openshift UI](./images/ArgoCD-Link.png) navigate to the cluster Argo CD link and login with the password above and admin.

3. create a folder config and copy utherp0's console link [example](./config/console-link.yaml) and commit.

4. In ArgoCD create an app:  

![](images/ArgoCD-Config.png)

This creates a nice App in the UI:

![](images/ArgoCD-App.png)

## Trying to create a simple pipeline using Tekton and ArgoCD

1. created a pipeline directory to save work and an ArgoCD app: ![](images/ArgoCD-Pipeline-App.png)

2. 


Reminder: need to point at the correct kubeconfig:

```shell
export KUBECONFIG=/Users/edoatley/code/openshift/auth/kubeconfig
```