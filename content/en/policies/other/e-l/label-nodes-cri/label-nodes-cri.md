---
title: "Label Nodes with CRI Runtime"
category: other
version: 1.7.0
subject: Node, Label
policyType: "mutate"
description: >
    CRI engines log in different formats. Loggers deployed as DaemonSets don't know which format to apply because they can't see this information. By Kyverno writing a label to each node with its runtime, loggers can use node label selectors to know which parsing logic to use. This policy detects the CRI engine in use and writes a label to the Node called `runtime` with it. The Node resource filter should be removed and users may need to grant the Kyverno ServiceAccount permission to update Nodes.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/e-l/label-nodes-cri/label-nodes-cri.yaml" target="-blank">/other/e-l/label-nodes-cri/label-nodes-cri.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: label-nodes-cri
  annotations:
    policies.kyverno.io/title: Label Nodes with CRI Runtime
    policies.kyverno.io/category: other
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Node, Label
    kyverno.io/kyverno-version: 1.7.2
    policies.kyverno.io/minversion: 1.7.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >-
      CRI engines log in different formats. Loggers deployed as DaemonSets don't know
      which format to apply because they can't see this information. By Kyverno writing a label
      to each node with its runtime, loggers can use node label selectors to know which parsing logic to use.
      This policy detects the CRI engine in use and writes a label to the Node called `runtime` with it.
      The Node resource filter should be removed and users may need to grant the Kyverno ServiceAccount permission
      to update Nodes.
spec:
  mutateExistingOnPolicyUpdate: true
  rules:
    - name: label-node-containerd
      match:
        any:
        - resources:
            kinds:
            - Node
      mutate:
        targets:
        - apiVersion: v1
          kind: Node
          name: "{{ request.object.metadata.name }}"
        patchStrategicMerge:
          metadata:
            labels:
              runtime: containerd
          status:
            nodeInfo:
              <(containerRuntimeVersion): containerd*
    - name: label-node-docker
      match:
        any:
        - resources:
            kinds:
            - Node
      mutate:
        targets:
        - apiVersion: v1
          kind: Node
          name: "{{ request.object.metadata.name }}"
        patchStrategicMerge:
          metadata:
            labels:
              runtime: docker
          status:
            nodeInfo:
              <(containerRuntimeVersion): docker*
```
