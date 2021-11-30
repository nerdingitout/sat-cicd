# Build CI/CD Pipeline across multiple Red Hat OpenShift Clusters
This guide will focus on setting up the pipeline across different clusters on the Cloud and Satellite.
### Architecture Diagram
![sat](https://user-images.githubusercontent.com/36239840/144006939-4c3d94e8-5715-492f-9b6a-17a0a3733fb7.png)

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
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/location%20b/pipeline-starter-clusterrole.yaml -n prod-env
```
```
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/location%20b/pipeline-starter-rolebinding.yaml -n prod-env
```
- Obtain the ```pipeline-starter```authentication token
```
oc sa get-token pipeline-starter -n prod-env
```
#### Setting up the pipeline in Production Environment
- Create tasks
```
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/location%20b/remote-task.yaml -n prod-env
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/tasks/apply-manifest-task.yaml -n prod-env
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/tasks/test-task.yaml -n prod-env
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/tasks/update-deployment-task.yaml -n prod-env
```
- Create Pipeline
```
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/location%20b/pipeline-b.yaml -n prod-env
```
- Create PVC
<br>From the Administrator perspective on the web console, go to storage and access PersistentVolumeClaims section. Click Create Persistent Volume Claim. Fill in the details as shown in the screenshot below.<br>
![image](https://user-images.githubusercontent.com/36239840/144010178-f6296011-7f0a-4fe9-b1ae-f7102b05a264.png)

### Location A - Dev Environment
- Create ```dev-env``` project
```
oc new-project dev-env
```
- Create ```pipeline-starter``` secret in ```dev-env``` project to access the ```prod-env``` project
```
oc create secret generic --from-literal=openshift-token=INSERT_TOKEN_HERE pipeline-starter -n dev-env
```
#### Setting up the pipeline in Development Environment
- Create tasks
```
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/location%20a/execute-remote-pipeline-task.yaml -n dev-env
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/tasks/apply-manifest-task.yaml -n dev-env
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/tasks/test-task.yaml -n dev-env
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/tasks/update-deployment-task.yaml -n dev-env
```
- Create Pipeline
```
oc create -f https://raw.githubusercontent.com/nerdingitout/sat-cicd/main/location%20a/pipeline-a.yaml -n dev-env
```
- Create PVC
<br>From the Administrator perspective on the web console, go to storage and access PersistentVolumeClaims section. Click Create Persistent Volume Claim. Fill in the details as shown in the screenshot below.<br>
![image](https://user-images.githubusercontent.com/36239840/144009663-35e70b43-0ee0-4b12-b1e4-04e7decd11f3.png)

- Make sure to edit the ```openshift-server-url``` parameter in ```exectue-remote-pipeline``` task in the pipeline yaml (line 101). Add the remote cluster URL (Location B) to connect to it. (the following lines for reference)
```
- name: execute-remote-pipeline
    params:
    - name: APP_NAME
      value: $(params.deployment-name)
    - name: url
      value: $(params.git-url)
    - name: pipeline-name
      value: prod-pipeline
    - name: pipeline-namespace
      value: prod-env
    - name: openshift-server-url
      value: INSERT_OPENSHIFT_URL_HERE
    - name: openshift-token-secret
      value: pipeline-starter
 ```
### Resources
- https://piotrminkowski.com/2021/08/05/kubernetes-ci-cd-with-tekton-and-argocd/
- https://dzone.com/articles/cicd-pipeline-spanning-multiple-openshift-clusters
- https://github.com/noseka1/execute-remote-pipeline
- https://containerjournal.com/features/standardizing-multi-cloud-k8s-deployments-with-tekton/
