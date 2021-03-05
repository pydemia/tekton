# Tekton Pipelines: Tutorials

## Scenario

There is an spring-boot app called `springboot-demo`.
Create an CI/CD pipeline to build a container image, triggered by Github Pull request in `master`.

## Prerequisites

* k8s Cluster
* Github
  * Account and target repository
  * **webhook**
* Container Registry(Harbor)
* Private Storage(MinIO)
* **Authorities**
  * `secret` for Github token
  * `secret` for private registry
  * (Optional) `secret` for private storage
  * `serviceAccount` with `clusterrole` and `clusterrolebinding`

### Github Webhook

https://github.com/pydemia/{registry_name}/settings/hooks

### Authorities

#### `secret` for Github token

https://github.com/settings/tokens

* repo
*  admin:repo_hook

```bash
github_personal_access_token_secret=""

kubectl -n default apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: github-token
type: Opaque
data:
  token: "$(echo $github_personal_access_token_secret|base64)" # in base64 encoded form
EOF

kubectl annotate secret github-token \
  -n default \
  tekton-pipeline.app='test'
```

#### `secret` for private registry

```bash
HARBOR_HOST="minio.airuntime.com"
HARBOR_USERNAME=""
HARBOR_PASSWORD=""
HARBOR_EMAIL=""
kubectl create secret docker-registry harbor-cred \
  -n default \
  --docker-server=${HARBOR_HOST} \
  --docker-username=${HARBOR_USERNAME} \
  --docker-password=${HARBOR_PASSWORD} \
  --docker-email=${HARBOR_EMAIL}

kubectl annotate secret harbor-cred \
  -n default \
  tekton-pipeline.app='test'
```

#### `secret` for private storage

```yaml
```

<!-- 
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: minio-cred
  annotations:
     serving.kubeflow.org/s3-endpoint: <minio-s3-compatible-endpoint> # replace with your s3 endpoint
     serving.kubeflow.org/s3-usehttps: "1" # by default 1, for testing with minio you need to set to 0
     serving.kubeflow.org/s3-verifyssl: "1" # by default 1, for testing with minio you need to set to 0
    #  serving.kubeflow.org/s3-region: us-east-1
type: Opaque
data:
  # echo -ne "KEY_STRING" | base64
  awsAccessKeyID: AKAKAKAKAKAKAKKA=
  awsSecretAccessKey: cVkcVkcVkcVkcVkcVkcVkcVkcVkcVkcVkcVkcVkcVk==
``` -->

#### `clusterrole` for deployment

```yaml
kubectl create clusterrole tekton-pipeline-manager \
  -n default \
  --verb="*" \
  --resource=deployments,deployments.apps

kubectl annotate clusterrole tekton-pipeline-manager \
  -n default \
  tekton-pipeline.app='test'
```

or 

```yaml
kubectl -n default apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: tekton-pipeline-manager
rules:
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - '*'
EOF

kubectl annotate clusterrole tekton-pipeline-manager \
  -n default \
  tekton-pipeline.app='test'
```

#### Create `serviceAccount` with authorities

```yaml
kubectl -n default apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-pipeline-manager
  annotations:
    tekton-pipeline.app: test
secrets:
  - name: harbor-cred
  - name: minio-cred
EOF
```


```bash
kubectl create clusterrolebinding tekton-pipeline-manager \
  -n default \
  --clusterrole=tekton-pipeline-manager \
  --serviceaccount=default:tekton-pipeline-manager

kubectl annotate clusterrolebinding tekton-pipeline-manager \
  -n default \
  tekton-pipeline.app='test'
```

---

### Resources

#### PullRequestResource

```yaml
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: springboot-demo-pr
spec:
  type: pullRequest
  params:
    - name: url
      value: https://github.com/pydemia/springboot-demo/pulls/1
    # - name: provider
    #   value: github
  # secret or use the above provider option
  secrets:
    - fieldName: authToken
      secretName: github-token
      secretKey: token
```

#### ImageResource

```yaml
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: springboot-base
spec:
  type: image
  params:
    - name: url
      value: docker.io/azul/zulu-openjdk-alpine:14
```

### Task

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: build-docker-image-from-git-source
spec:
  params:
    - name: pathToDockerFile
      type: string
      description: The path to the dockerfile to build
      default: $(resources.inputs.docker-source.path)/Dockerfile
    - name: pathToContext
      type: string
      description: |
        The build context used by Kaniko
        (https://github.com/GoogleContainerTools/kaniko#kaniko-build-contexts)
      default: $(resources.inputs.docker-source.path)
  resources:
    inputs:
      - name: docker-source
        type: git
    outputs:
      - name: builtImage
        type: image
  steps:
    - name: build-and-push
      image: gcr.io/kaniko-project/executor:v0.16.0
      # specifying DOCKER_CONFIG is required to allow kaniko to detect docker credential
      env:
        - name: "DOCKER_CONFIG"
          value: "/tekton/home/.docker/"
      command:
        - /kaniko/executor
      args:
        - --dockerfile=$(params.pathToDockerFile)
        - --destination=$(resources.outputs.builtImage.url)
        - --context=$(params.pathToContext)
```

### TaskRun

```yaml
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: build-docker-image-from-git-source-task-run
spec:
  serviceAccountName: tekton-pipeline-manager
  taskRef:
    name: build-docker-image-from-git-source
  params:
    - name: pathToDockerFile
      value: Dockerfile
    - name: pathToContext
      value: $(resources.inputs.docker-source.path)/examples/microservices/leeroy-web #configure: may change according to your source
  resources:
    inputs:
      - name: docker-source
        resourceRef:
          name: skaffold-git
    outputs:
      - name: builtImage
        resourceRef:
          name: skaffold-image-leeroy-web
```

