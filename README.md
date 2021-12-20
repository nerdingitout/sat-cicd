# Build CI/CD Pipeline across multiple Red Hat OpenShift Clusters
This guide will focus on setting up the pipeline across different clusters on the Cloud and Satellite.
## Prerequisites 
- For each classic virtual server (worker nodes) for the openshift cluster, it should have minimum 4vCPU, 16 GB RAM, 3 disks (100 GB for boot, at least 25 GB for /var/data disk, a free unmounted and unpartitioned disk can be 100 GB).
- Shared Storage for the pipeline. Check: <a href="https://cloud.ibm.com/docs/satellite?topic=satellite-config-storage-local-file">Setting up local file storage on Red Hat OpenShift on IBM Cloud Satellite</a>
## Architecture Diagram
![sat](https://user-images.githubusercontent.com/36239840/144072701-4de95c75-9b9b-495f-8fb1-e40723ce4a93.png)

## Location B - Production Environment
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
### Setting up the pipeline in Production Environment
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

## Location A - Dev Environment
- Create ```dev-env``` project
```
oc new-project dev-env
```
- Create ```pipeline-starter``` secret in ```dev-env``` project to access the ```prod-env``` project
```
oc create secret generic --from-literal=openshift-token=INSERT_TOKEN_HERE pipeline-starter -n dev-env
```
### Setting up the pipeline in Development Environment
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
## Trigger the Pipeline
- Run the following command to trigger the pipeline that starts from location A which in turn triggers the pipeline in location B. Make sure to change the values indicated below according to your project, pipeline and deployment details.
```
tkn pipeline start <pipeline-name> -w name=shared-workspace,ClaimName=source-pvc -p deployment-name=<deployment-name>
    -p git-url=<git-url> --use-param-defaults
```
## Connect to Postgres DB
- This step is to be applied once and only for the backend application <a href="https://github.com/nerdingitout/form-bff">form bff</a>. Make sure to connect to your postgres database by creating a secret in each environment using the following command. Make sure to replace the values wit the right credentials for each variable. Leave port 8080 as is.<br>
```
oc create secret generic postgredb-secret --from-literal=DB_USER=<add-db-user-here> --from-literal=DB_PASSWORD=<add-db-password-here> --from-literal=DB_HOST=<add-db-host-here> --from-literal=DB_PORT=<add-db-port-here> --from-literal=DB_NAME=<add-db-name-here> --from-literal=PORT=8080
```
- Then set the secret you create as environment variable for your application<br>
```
oc set env --from=secret/postgredb-secret deployment/form-bff
```
## Resources
- https://piotrminkowski.com/2021/08/05/kubernetes-ci-cd-with-tekton-and-argocd/
- https://dzone.com/articles/cicd-pipeline-spanning-multiple-openshift-clusters
- https://github.com/noseka1/execute-remote-pipeline
- https://containerjournal.com/features/standardizing-multi-cloud-k8s-deployments-with-tekton/
- https://cloud.ibm.com/docs/satellite?topic=satellite-hosts
