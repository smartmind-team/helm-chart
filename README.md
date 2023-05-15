# __Helm-Chart__
This repository is a collection of Smartmind's Helm charts. Currently there are two helm charts in this repository: `ecr-secrets` and `thanosql-engine`. 
- __ecr-secrets__: The ecr-secrets is a helm-chart that hosts a cron task that authenticates the aws-cli using appropriate AWS credentials. The reason behind this is to make sure that the kubernetes pods are able to pull private images without the developer needing to manually create a secret every 12 hours. 
- __thanosql-engine__: This Helm chart builds the [ThanoSQL-Engine](https://github.com/smartmind-team/thanosql-engine) as a Kubernetes `deployment` using appropriate configurations using the values provided.

This repo also has a github action that runs whenever a release is made. This action automatically builds the helm charts and hosts them privately using Smartmind's github user content. The details of adding and using the helm charts are listed below. 


## __Adding New Charts__
After testing the helm chart on your system and you are certain that it has no errors, create a new branch with the correct version in your `Chart.yaml` and commit. 

Then make a pull request into main. After getting reviewed merge into main. This results in the `github action` running to create a `.tgz` in the gh-pages branch. The github pages then can be used by helm to pull from this private github repository.

## __Using the Helm Charts__
The helm charts are hosted in the gh-pages of this repository. Proceed with the following commands to utilize the helm charts.

__To Download the Helm Repo__
```bash
helm repo add {release-name-to-save-as} https://raw.githubusercontent.com/smartmind-team/helm-chart/gh-pages --username {github-username} --password {github-access-token}

# check to see if the chart was added
helm repo list
```
The above command pulls the repository from the gh-pages of this repository. In order to do that, the puller should have access to this private repository (i.e. being a member of SmartMind).

__To Run the Helm Chart__
```
helm upgrade --install {release-name} {chart-name/chart-name} --namespace {chosen-namespace} --values {config.yaml}
```
This allows you to install the helm chart from your helm repository list. The `config.yaml` should contain the values to inject into the helm chart. 

## __Charts and their Values__
### __thanosql-engine__
__Chart__

This is a helm chart that is used to deploy the [thanosql-engine](https://github.com/smartmind-team/thanosql-engine) on kubernetes.

The templates consist of a configmap (passes in the environment values to the engine), a service (enables to port forward the engine so that users can access via API), and a deployment (runs the actual engine image).

__Values Needed__

```yaml
# config.yaml
replicaCount: 

image:
  repository: 
  pullPolicy: 
  tag: 
  port: 
  command: 

imagePullSecrets:
  - name: 

service:
  type: 
  port: 
  loadBalancerIP: 
  clusterIP: 

resources:
  # requests and limits needed for the engine go here

configmap:
  data:
    # ENV variables required for the Engine goes here

volumes:
  - name: 
    hostPath:
      path: 
      type: 
  - name: 
    emptyDir:
      medium: 
      sizeLimit: 

volumeMounts:
  - name: 
    mountPath: 
  - name: 
    mountPath: 
```

### __ecr-secret__
__Chart__

This is a helm chart that allows the Amazon AWS ECR Token to refresh consistently. This was implemented since the ECR Token only lasts for 12 hours, resulting in manual creation and deletion of Kubernetes Secret.

This helm chart mainly runs a cronjob that automates the deletion and creation of the ECR token using a combitnation of secret, service account, configmap, cronjob, role, and rolebinding. 

__Values Needed__

```yaml
# config.yaml
secret:
  stringData:
    AWS_SECRET_ACCESS_KEY: 
    AWS_ACCESS_KEY_ID: 
    AWS_ACCOUNT: 

configmap:
  data:
    AWS_REGION:
    DOCKER_SECRET_NAME: 

cronjob:
  schedule: 
```



## References

- [chart-releaser github action](https://github.com/helm/chart-releaser-action)
- [chart-releaser demo](https://github.com/helm/charts-repo-actions-demo)
