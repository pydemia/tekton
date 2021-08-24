# Tekton Dashboard

https://github.com/tektoncd/dashboard
https://tekton.dev/docs/dashboard/


## Installation

https://github.com/tektoncd/dashboard/blob/main/docs/install.md

### Install Tekton Pipeline

```bash
# latest
$ tekton_dashboard_url="https://storage.googleapis.com/tekton-releases/dashboard/latest/tekton-dashboard-release.yaml"
# latest for now: v0.14.0(on 20210307)
$ tekton_dashboard_url="https://storage.googleapis.com/tekton-releases/dashboard/previous/v0.14.0/tekton-dashboard-release.yaml"


$ curl -fsSL $tekton_dashboard_url -o tekton-dashboard-release.yaml && \
    kubectl apply -f tekton-dashboard-release.yaml \
    > install_tekton_dashboard.log 2>&1
```

Monitor the installation

```bash
$ kubectl -n tekton-pipelines get pods --watch

NAME                                           READY   STATUS    RESTARTS   AGE
tekton-dashboard-74f599b484-99gxf              1/1     Running   0          30s
tekton-pipelines-controller-8677dbffd6-r6wr5   1/1     Running   0          19d
tekton-pipelines-webhook-f697fd54c-vljs9       1/1     Running   0          19d
```

### Access to the dashboard

```bash
kubectl -n tekton-pipelines port-forward svc/tekton-dashboard 9097:9097
```

### Expose the dashboard

```