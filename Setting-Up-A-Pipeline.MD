Setting up a sample OCP pipeline
================================

## Initial set up

1. Spin up OpenShift Cluster
2. Install Pipelines Operator

## SQ install

```shell
oc new-project sonarqube
Now using project "sonarqube" on server "https://api.test1.edoatley.com:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=k8s.gcr.io/serve_hostname
$ oc apply -f sonarqube/sonar-template.yaml
W0924 09:11:08.264788   24183 shim_kubectl.go:55] Using non-groupfied API resources is deprecated and will be removed in a future release, update apiVersion to "template.openshift.io/v1" for your resource
template.template.openshift.io/sonarqube created
$ oc process -n sonarqube sonarqube | oc create -f -                                                   Py base ○ sonarqube/api-test1-edoatley-com:6443/system:admin/sonarqube 09:16:57
service/sonarqube created
route.route.openshift.io/sonarqube created
imagestream.image.openshift.io/sonarqube created
deploymentconfig.apps.openshift.io/sonarqube created
service/postgresql-sonarqube created
deploymentconfig.apps.openshift.io/postgresql-sonarqube created
persistentvolumeclaim/postgresql-sonarqube-data created
persistentvolumeclaim/sonarqube-data created
persistentvolumeclaim/sonarqube-extensions created
persistentvolumeclaim/sonarqube-logs created
persistentvolumeclaim/sonarqube-temp created
$ oc get routes -n sonarqube                                                                       15s Py base ○ sonarqube/api-test1-edoatley-com:6443/system:admin/sonarqube 09:18:14
NAME        HOST/PORT                                     PATH   SERVICES    PORT    TERMINATION   WILDCARD
sonarqube   sonarqube-sonarqube.apps.test1.edoatley.com          sonarqube   <all>   edge          None
```

These links were useful:
- [OCP templates](https://docs.openshift.com/container-platform/4.2/openshift_images/using-templates.html#templates-creating-from-console_using-templates)
- [SQ on OCP](https://dale-bingham-cingulara.medium.com/setup-sonarqube-in-minishift-for-scanning-projects-through-jenkins-a70a6e2d93d3)

sq token:

0702871372ea80188821963e67ceba83c56c6633