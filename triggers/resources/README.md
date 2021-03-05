# Tekton Triggers: Resources

https://github.com/tektoncd/triggers/blob/release-v0.11.x/docs/triggertemplates.md

The resources of Tekton triggers:

* TriggerTemplate
  > It is a resource that template resources. It has parameters that can be substituted anywhere within the resource template.
  > | v1alpha1 | v1beta1 |
  > | :------- | :------ |
  > | pipelines | pipelines |
  > | pipelineruns | pipelineruns |
  > | tasks | tasks |
  > | taskruns | taskruns |
  > | clustertasks | clustertasks |
  > | conditions | - |
  > | pipelineresources | - |
  > If the namespace is omitted, it will be resolved to the `EventListener`'s namespace.
  > `params` can be referenced in the TriggerTemplate using the following variable substitution syntax, where `<name>` is the name of the parameter:
  > ```bash
  > $(tt.params.<name>)
  > ```
  > `tt.params` can be referenced in the `resourceTemplates` section of a `TriggerTemplate`. The purpose of tt.params is to make `TriggerTemplates` reusable.
* TriggerBinding
  > It binds against events/triggers and enable you to capture fields from an event and store them as parameters.
  > The separation of `TriggerBindings` from `TriggerTemplates` was deliberate to encourage reuse between them.
  > `TriggerBindings` are connected to `TriggerTemplates` within an `EventListener`, which is where the pod is actually instantiated that "listens" for the respective events.
* EventListener
  > It allows users a declarative way to process incoming HTTP based events with JSON payloads.
  > `EventListeners` expose an addressable "Sink" to which incoming events are directed. Users can declare `TriggerBindings` to extract fields from events, and apply them to `TriggerTemplates` in order to create Tekton resources. In addition, `EventListeners` allow lightweight event processing using Event `Interceptors`.
  * Interceptors
    * Webhook Interceptors
    * GitHub Interceptors
    * GitLab Interceptors
    * Bitbucket Interceptors
    * CEL Interceptors
    > `Triggers` within an `EventListener` can optionally specify interceptors, to modify the behavior or payload of `Triggers`.
* ClusterTriggerBinding
  > It is similar to `TriggerBinding` which is used to extract field from event payload by used in `EventListeners`.
  > `ClusterTriggerBinding` is designed as cluster-scoped to encourage reusability clusterwide.
