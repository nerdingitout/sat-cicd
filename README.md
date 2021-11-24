# Build CI/CD Pipeline across multiple Red Hat OpenShift Clusters
This guide will focus on setting up the pipeline across different clusters on the Cloud and Satellite.
### Architecture Diagram
![sat-cicd](https://user-images.githubusercontent.com/36239840/143217030-5801a6c1-8036-49dc-b51d-8ee8b01c413d.png)

### Location B - Production Environment
- Create ```prod-env``` project
```
oc new-project prod-env
```
- Create Service account and give it the right permissions 
```
oc create sa pipeline-starter -n prod-env
```
```
oc create -f -n prod-env
```
```
oc create -f -n prod-env
```
- Obtain the ```pipeline-starter```authentication token
```
oc sa get-token pipeline-starter -n prod-env
```

### Location A - Dev Environment
- Create ```dev-env``` project
```
oc new-project dev-env
```
- Create ```pipeline-starter``` secret in ```dev-env``` project to access the ```prod-env``` project
```
oc create secret generic --from-literal=openshift-token=INSERT_TOKEN_HERE pipeline-starter -n dev-env
```

### Resources
- https://piotrminkowski.com/2021/08/05/kubernetes-ci-cd-with-tekton-and-argocd/
- https://dzone.com/articles/cicd-pipeline-spanning-multiple-openshift-clusters
- https://github.com/noseka1/execute-remote-pipeline
- https://containerjournal.com/features/standardizing-multi-cloud-k8s-deployments-with-tekton/
