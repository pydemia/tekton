# Tekton Pipelines: Resources

The resources of Tekton pipelines:

* PipelineResource 
  > The set of objects that are going to be used as inputs and outputs of `Task`.
  > It's type is defiend in `spec.type` in yaml(ex. `spec.type: git`), and it is used by `spec.resources` in `Task`, specifying its `name` and `type`.)
  * Git Resource
  * PullRequest Resource
  * Image Resource
  * Storage Resource
  * CloudEvent Resource
  * Cluster Resource
* Task & TaskRun
  * Task
  > A Series of `steps` that runs in a desired order and complete a set amount of build work. `spec.resources` in `Task` defines logical names and types of resources, not the exact ones defined as `PipelineResources`.
  > It contains `steps` commands, Def. of `params` and `resources` as placeholders.
    * `Task`: run as a Pod
    * `step`: run as a container in the `Task` pod.
  * TaskRun: Instantiate a Task
    > It binds the inputs and outputs in `Task` to already defined `PipelineResources`
    > It contains the values of `params`, `resources` bounded by `resourceRef`, and `serviceAccountName` which has proper authorities with keys and tokens for docker registry, storage, etc.
* Pipeline & PipelineRun
  * Pipeline
    > An ordered series of `Tasks` that you want to execute along with the corresponding inputs and outputs for each `Task`, defining it in `spec.tasks`. `Task` is ordered by  `from` property in `spec.tasks.resources.inputs`.
    > It contains `tasks` referencing `Tasks` by `taskRef`, `resources` that centralized definitions called by each `tasks`. the values of `params` in `tasks` are given but `resourceDef` is not.
  * PipelineRun: Instantiate a Pipeline
    > It contains `serviceAccountName`, `pipelineRef`, `resources` bounded by `resourceRef`.
    > `serviceAccount` has proper authorities and `clusterrolebinding` of `verb=*, resource=deployments,deployments.apps`.
  * 

---
