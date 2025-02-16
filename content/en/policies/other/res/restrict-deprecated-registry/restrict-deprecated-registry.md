---
title: "Restrict Deprecated Registry"
category: Best Practices, EKS Best Practices
version: 1.9.0
subject: Pod
policyType: "validate"
description: >
    Legacy k8s.gcr.io container image registry will be frozen in early April 2023 k8s.gcr.io image registry will be frozen from the 3rd of April 2023.   Images for Kubernetes 1.27 will not be available in the k8s.gcr.io image registry. Please read our announcement for more details. https://kubernetes.io/blog/2023/02/06/k8s-gcr-io-freeze-announcement/     
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/res/restrict-deprecated-registry/restrict-deprecated-registry.yaml" target="-blank">/other/res/restrict-deprecated-registry/restrict-deprecated-registry.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-deprecated-registry
  annotations:
    policies.kyverno.io/title: Restrict Deprecated Registry
    policies.kyverno.io/category: Best Practices, EKS Best Practices
    policies.kyverno.io/severity: high
    policies.kyverno.io/minversion: 1.9.0
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Legacy k8s.gcr.io container image registry will be frozen in early April 2023
      k8s.gcr.io image registry will be frozen from the 3rd of April 2023.  
      Images for Kubernetes 1.27 will not be available in the k8s.gcr.io image registry.
      Please read our announcement for more details.
      https://kubernetes.io/blog/2023/02/06/k8s-gcr-io-freeze-announcement/     
spec:
  validationFailureAction: Enforce
  # validationFailureAction: Audit
  background: true
  rules:
  - name: restrict-deprecated-registry
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
        message: "The \"k8s.gcr.io\" image registry is deprecated. \"registry.k8s.io\" should now be used."
        foreach:
          - list: "request.object.spec.[initContainers, ephemeralContainers, containers][]"
            deny:
              conditions:
                all:
                  - key: "{{ element.image }}"
                    operator: Equals
                    value: "k8s.gcr.io/*"

```
