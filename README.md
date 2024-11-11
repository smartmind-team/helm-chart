# __Helm-Chart__
This repository is a collection of Smartmind's Helm charts. Currently there are five helm charts in this repository: `aws-thanosql`, `ecr-secrets`, `file-manager`, `thanosql-engine`, `thanosql-engine-async`. 
- __aws-thanosql__: The aws-thanosql is a helm-chart that is used to spawn an aws thanosql workspace. It consists of three applications: [thanosql-api](https://github.com/smartmind-team/thanosql-api), [thanosql](https://github.com/smartmind-team/thanosql), and the [single user image](https://github.com/smartmind-team/thanosql-singleuser).
- __ecr-secrets__: The ecr-secrets is a helm-chart that hosts a cron task that authenticates the aws-cli using appropriate AWS credentials. The reason behind this is to make sure that the kubernetes pods are able to pull private images without the developer needing to manually create a secret every 12 hours. This chart is used only for the rack server kubernetes.
- __file-manager__: This Helm chat builds the [file manager](https://github.com/smartmind-team/workspace-fm-be) application that is used to manage the files in the user's workspace. This repository is deprecated in the AWS ThanoSQL and is only used in older versions of the Rack Server ThanoSQL. 
- __thanosql-engine__: This Helm chart builds the [ThanoSQL-Engine](https://github.com/smartmind-team/thanosql-engine) as a Kubernetes `deployment` using appropriate configurations using the values provided. This repository is mostly deprecated and not used.
- __thanosql-engine-async__: This Helm chart builds an async version of [ThanoSQL-Engine](https://github.com/smartmind-team/thanosql-engine/pull/287). This pull request was never merged as it was not seen as immediately necessary, hence, the chart is not be valid as well. 

This repo also has a github action that runs whenever a release is made. This action automatically builds the helm charts and hosts them privately using Smartmind's github user content. The details of adding and using the helm charts are listed below. 


## __Adding New Charts__
After testing the helm chart on your system and you are certain that it has no errors, create a new branch with the correct version in your `Chart.yaml` and commit. 

After making a pull request into main create a new release. This results in the `github action` running to create a `.tgz` in the gh-pages branch. The github pages then can be used by helm to pull from this private github repository.

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
### __aws_thanosql__
__Chart__

The aws thanosql helm chart, deploys all of the required components to make up a ThanoSQL workspace. This consists of the `thanosql` engine, `thanosql-api`, and the `singleuser` image, where each of the components consist of a Kubernetes `service`, `deployment`, and `config map`.

This helm chart is normally deployed using the [thanosql-launcher](https://github.com/smartmind-team/thanosql-launcher) since most of the values will be fixed. Here is the `values.yaml` regardless:

__Values Needed__

```yaml
# Default values for aws-thanosql.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

nodeName: #
storage: 
storageClass:
certificateArn:

jupyterlab:
  replicaCount: 

  image:
    repository: 
    pullPolicy: 
    tag: 
    port: 
    command: |

  service:
    type: 
    port: 

  configmap:
    data:
      FRAME_ANCESTOR: 
      NVIDIA_VISIBLE_DEVICES: 

engine:
  replicaCount: 

  image:
    repository: 
    pullPolicy: 
    tag: 
    command: |

  service:
    type: 
    port: 

  configmap:
    data:
      POSTGRES_PASSWORD: 
      PG_USER: 
      PG_PASSWORD: 
      DAYS: 
      CRON: 
      PYTHONPATH: 
      THANOSQL_TOKEN: 
      NVIDIA_DRIVER_CAPABILITIES:
      NVIDIA_VISIBLE_DEVICES: 
      AWS_SHARED_CREDENTIALS_FILE: 

api:
  replicaCount: 
  image:
    repository: 
    pullPolicy: 
    tag: 
    port:
    args: |

  service:
    type: 
    port: 

  configmap:
    data:
      AUTHJWT_SECRET_KEY: 
      AUTHJWT_ALGORITHM: 
      ADMIN_USER: 
      ADMIN_PASSWORD: 
      MODEL_BASE_DIR: 
      POSTGRES_SERVER: 
      POSTGRES_DB: 
      POSTGRES_USER: 
      POSTGRES_PASSWORD: 
      ENGINE_ENV: 
      CORS_ORIGIN: 
      THANOSQL_PASSWORD: 
      THANOSQL_USER: 
      THANOSQL_DB: 
      USERID: 
      USERHOME: 
      USERNAME: 
      WORKSPACE_ID: 
      VOLUME_MAX_SIZE: 
      NVIDIA_VISIBLE_DEVICES: 
      HOME: 
```


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

### __file-manager__
__Chart__
The file manager allows the users to manage their files in their workspace's file system. This chart is deprecated in the AWS version as it's functionalities were merged into the [thanosql api](https://github.com/smartmind-team/thanosql-api) repository.

__Values Needed__
```yaml
replicaCount: 

image:
  repository: 
  pullPolicy: 
  # Overrides the image tag whose default is the chart appVersion.
  tag: 
  port:
imagePullSecrets:
  - name: 

service:
  type: 
  port: 
  loadBalancerIP: 

resources:
  limits:
    cpu: 
    memory:

configmap:
  data:
    DB_USERNAME: 
    DB_PASSWORD: 
    DB_HOST: 
    DB_PORT: 
    DB_NAME: 
    AUTHJWT_SECRET_KEY: 
    AUTHJWT_ALGORITHM: 
    VOLUME_MAX_SIZE: 
    CHANGE_USER: 
    CHANGE_UID: 
    USER_DATA_ROOT: 
    CHANGE_HOME: 
    USR: 
    WORKSPACE_ID: 


volumeMounts:
  - name: 
    mountPath: 

volumes:
  - name: 
    persistentVolumeClaim:
      claimName: 
```

## References

- [chart-releaser github action](https://github.com/helm/chart-releaser-action)
- [chart-releaser demo](https://github.com/helm/charts-repo-actions-demo)
