# Tekton Pipeline

https://github.com/tektoncd/pipeline
https://tekton.dev/docs/pipelines/


## Installation

https://github.com/tektoncd/pipeline/blob/master/docs/install.md

### Prerequisite

* Kubernetes cluster running version 1.16 or later
* (Optional) Set Kubernetes Metric Server
  - ```bash
    curl -fsSL https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml -o k8s-metrics-server-components.yaml && \
    kubectl apply -f k8s-metrics-server-components.yaml \
    > install_k8s_metrics_server.log 2>&1

    # serviceaccount/metrics-server (label configured)
    # clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader (created)
    # clusterrole.rbac.authorization.k8s.io/system:metrics-server (modified)
    # rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader (label configured)
    # clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator (label configured)
    # clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server (label configured)
    # service/metrics-server (label configured)
    # deployment.apps/metrics-server (created)
    # apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io (label configured)
    ```

### Install Tekton Pipeline

```bash
# latest
$ tekton_pipelines_url="https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml"
# latest for now: v0.20.1(on 20210128)
$ tekton_pipelines_url="https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.20.1/release.yaml"


$ curl -fsSL $tekton_pipelines_url -o tekton-pipelines-release.yaml && \
    kubectl apply -f tekton-pipelines-release.yaml \
    > install_tekton_pipelines.log 2>&1
```

Monitor the installation

```bash
$ kubectl -n tekton-pipelines get pods --watch

NAME                                           READY   STATUS    RESTARTS   AGE
tekton-pipelines-controller-8677dbffd6-2f5zc   1/1     Running   0          72s
tekton-pipelines-webhook-f697fd54c-s5fz4       1/1     Running   0          69s
```

### Configuring the Bucket

```bash
kubectl -n tekton-pipelines create secret generic harbor-cert --from-file=ca.crt=./harbor-ca.crt
```

```yaml
AWS_ACCESS_KEY_ID="<your-key-id>"
AWS_SECRET_ACCESS_KEY="<your-access-key>"
HARBOR_CERT="$(kubectl -n tekton-pipelines get secrets harbor-cert -o jsonpath="{.data.ca\.crt}"|base64 -d)"
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: tekton-storage
  namespace: tekton-pipelines
type: kubernetes.io/opaque
stringData:
  boto-config: |
    [Credentials]
    aws_access_key_id = ${AWS_ACCESS_KEY_ID}
    aws_secret_access_key = ${AWS_SECRET_ACCESS_KEY}
    [s3]
    host = minio.airuntime.com
    [Boto]
    https_validate_certificates = True
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-artifact-bucket
  namespace: tekton-pipelines
data:
  location: s3://tekton-pipelines
  bucket.service.account.secret.name: tekton-storage
  bucket.service.account.secret.key: boto-config
  bucket.service.account.field.name: BOTO_CONFIG
# ---
# apiVersion: v1
# kind: ConfigMap
# metadata:
#   name: config-defaults
#   namespace: tekton-pipelines
#   labels:
#     app.kubernetes.io/instance: default
#     app.kubernetes.io/part-of: tekton-pipelines
# data:
#   default-cloud-events-sink: https://my-sink-url
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-registry-cert
  namespace: tekton-pipelines
  labels:
    app.kubernetes.io/instance: default
    app.kubernetes.io/part-of: tekton-pipelines
data:
  # Registry's self-signed certificate
  cert: "$(echo ${HARBOR_CERT}|base64)"
---
EOF
```

---

## Setup Task & Pipeline
