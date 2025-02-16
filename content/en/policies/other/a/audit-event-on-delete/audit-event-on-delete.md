---
title: "Audit Event on Delete"
category: Other
version: 1.10.0
subject: Secret
policyType: "generate"
description: >
    Kubernetes Events are limited in that the circumstances under which they are created cannot be changed and with what they are associated is fixed. It may be advantageous in many cases to augment these out-of-the-box Events with custom Events which can be custom designed to your needs. This policy generates an Event when a Secret has been deleted. It lists the userInfo of the actor performing the deletion.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/a/audit-event-on-delete/audit-event-on-delete.yaml" target="-blank">/other/a/audit-event-on-delete/audit-event-on-delete.yaml</a>

```yaml
apiVersion: kyverno.io/v2beta1
kind: ClusterPolicy
metadata:
  name: audit-event-on-delete
  annotations:
    policies.kyverno.io/title: Audit Event on Delete
    policies.kyverno.io/category: Other
    kyverno.io/kyverno-version: 1.10.0
    policies.kyverno.io/minversion: 1.10.0
    kyverno.io/kubernetes-version: "1.26"
    policies.kyverno.io/subject: Secret
    policies.kyverno.io/description: >-
      Kubernetes Events are limited in that the circumstances under which they are created
      cannot be changed and with what they are associated is fixed. It may be advantageous
      in many cases to augment these out-of-the-box Events with custom Events which can be
      custom designed to your needs. This policy generates an Event when a Secret has been
      deleted. It lists the userInfo of the actor performing the deletion.
spec:
  background: false
  rules:
  - name: generate-event-on-delete
    match:
      any:
      - resources:
          kinds:
          - Secret
          operations:
          - DELETE
    generate:
      apiVersion: v1
      kind: Event
      name: "delete.{{ random('[a-z0-9]{12}') }}"
      namespace: "{{request.object.metadata.namespace}}"
      synchronize: false
      data:
        firstTimestamp: "{{ time_now_utc() }}"
        involvedObject:
          apiVersion: v1
          kind: Secret
          name: "{{ request.name }}"
          namespace: "{{ request.namespace }}"
          uid: "{{request.oldObject.metadata.uid}}"
        lastTimestamp: "{{ time_now_utc() }}"
        message: The {{ request.name }} Secret was deleted by {{ request.userInfo | to_string(@) }}.
        reason: Delete
        source:
          component: kyverno
        type: Warning
```
