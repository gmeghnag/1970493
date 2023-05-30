# SRVKP-3140

## Issue

Resource limits and requests defined in Tekton `Task` [`{.spec.stepTemplate}`](https://tekton.dev/docs/pipelines/tasks/#specifying-a-step-template) are not applied to the related `Pod`'s `{.spec.initContainers}`.
  
## How-To-Reproduce
- Install OpenShift Pipelines Operator if not present:
  ```
  oc apply -k https://github.com/gmeghnag/openshift-pipelines
  ```
  
- Initiate the `Task` and `TaskRun` resources to reproduce the issue:
  ```
  oc apply -k https://github.com/gmeghnag/SRVKP-3140
  ```
  <details close>
  <summary><code>Task</code> definition</summary>
  <br>
  <pre>
  apiVersion: tekton.dev/v1beta1
  kind: Task
  metadata:
    name: hello-world
  spec:
    stepTemplate:
      resources:
        limits:
          cpu: "2"
          memory: 1Gi
        requests:
          cpu: "1"
          memory: 1Gi
    steps:
    - name: hello-world
      image: registry.access.redhat.com/ubi8/ubi:latest
      script: |
        echo "Hello world"
  </pre>
  <br><br>
  </details>
  <details close>
  <summary><code>TaskRun</code> definition</summary>
  <br>
  <pre>
  apiVersion: tekton.dev/v1beta1
  kind: TaskRun
  metadata:
    name: hello-world-run
  spec:
    taskRef:
      name: hello-world
  </pre>
  <br><br>
  </details>
  
## Diagnostic-Steps
Verify that the `resources.limits` and `resources.requests` are not applied to `initContainers` (but to `containers` only):
```
oc get po $(oc get taskrun hello-world-run -o jsonpath='{.status.podName}') -o json | jq '.spec | "initContainers.resources : " +  (.initContainers[0].resources|tostring) + "\ncontainers.resources     : " + (.containers[0].resources|tostring)' -r 
```
```
initContainers.resources : {}
containers.resources     : {"limits":{"cpu":"2","memory":"1Gi"},"requests":{"cpu":"1","memory":"1Gi"}}
```
