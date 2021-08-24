# Tekton Pipeline: Triggers

https://github.com/tektoncd/triggers
https://tekton.dev/docs/triggers/

Triggers enables users to map fields from an event payload into resource templates. Put another way, this allows events to both model and instantiate themselves as Kubernetes resources. In the case of `tektoncd/pipeline`, this makes it easy to encapsulate configuration into `PipelineRuns` and `PipelineResources`.

* TriggerTemplate
* TriggerBinding
* EventListener
* ClusterTriggerBinding

## Installation

https://github.com/tektoncd/triggers/blob/master/docs/install.md

```bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/previous/v0.1.0/release.yaml
```

### Prerequisite

* `k8s >= 1.17`
  * [Bump knative to release-0.20 and pipeline to v0.20.0](https://github.com/tektoncd/triggers/pull/903)
* `tekton-pipelines`
* Grant current-user `cluster-admin` privileges.

### Install Tekton Pipeline

```bash
$ tekton_triggers_url="https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml"
# latest for now: v0.11.2(on 20210128)
$ tekton_triggers_url="https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.11.2/release.yaml"

$ curl -fsSL $tekton_triggers_url -o tekton-triggers-release.yaml && \
    kubectl apply -f tekton-triggers-release.yaml \
    > install_tekton_triggers.log 2>&1

$ tekton_pipelines_url="https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.20.1/release.yaml"

$ curl -fsSL $tekton_pipelines_url -o tekton-pipelines-release.yaml && \
    kubectl apply -f tekton-pipelines-release.yaml \
    > reinstall_tekton_pipelines.log 2>&1
$ kubectl apply -f tekton-pipelines-release.yaml \
    > reinstall_tekton_pipelines-2.log 2>&1
```

> :warning::warning::warning: **Version Mismatching: Pipeline vs Triggers**
> Resources in `tekton-triggers-release.yaml` bumps to ones in `tekton-pipelines-release.yaml`, from its version. Using the same `pipelines` version (e.g. `pipeline=v0.11.2`) as `triggers` version has no problem.
> Mostly the bumps between them are **annotations** and some advances, except `clusterrole` & `clusterrolebinding` names, which are the only things that newly created; it does not exist in the higher `pipelines` version anymore.
> (e.g. `tekton-pipelines-controller-admin`: `v0.11.2` -> `tekton-pipelines-controller-cluster-access`: `v0.20.1`)
> If you want to use the higher `pipelines` version, you should re-install(overwrite) the `tekton-pipelines-release.yaml` again.
> **_When reinstalling, you would meet this error_**: `Internal error occurred: failed calling webhook "config.webhook.pipeline.tekton.dev": Post https://tekton-pipelines-webhook.tekton-pipelines.svc:443/config-validation?timeout=10s: no endpoints available for service "tekton-pipelines-webhook"`. Wait for restart the webhook and just do it again.


Monitor the installation

```bash
$ kubectl -n tekton-pipelines get pods --watch
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-pipelines-controller-8677dbffd6-r6wr5   1/1     Running   0          42m
tekton-pipelines-webhook-f697fd54c-vljs9       1/1     Running   0          42m
```

---

![[trigger-flow](https://tekton.dev/docs/triggers/)](TriggerFlow.png)