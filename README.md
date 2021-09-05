# gitops
PLace to play with RH Gitops in openshift

## References

This repository is inspired by [Dev Epihany Blog](https://devepiphany.org/2021/04/27/geitting-to-grips-with-gitops/)
This is useful info [Getting Started with OpenShift GitOps](https://github.com/siamaksade/openshift-gitops-getting-started)

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

Reminder: need to point at the correct kubeconfig:

```shell
export KUBECONFIG=/Users/edoatley/code/openshift/auth/kubeconfig
```