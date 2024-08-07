---
title: "Enforce ReadWriteOncePod in CEL expressions"
category: Sample in CEL
version: 1.11.0
subject: PersistentVolumeClaim
policyType: "validate"
description: >
    Some stateful workloads with multiple replicas only allow a single Pod to write to a given volume at a time. Beginning in Kubernetes 1.22 and enabled by default in 1.27, a new setting called ReadWriteOncePod, available for CSI volumes only, allows volumes to be writable from only a single Pod. For more information see the blog https://kubernetes.io/blog/2023/04/20/read-write-once-pod-access-mode-beta/. This policy enforces that the accessModes for a PersistentVolumeClaim be set to ReadWriteOncePod.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/enforce-readwriteonce-pod/enforce-readwriteonce-pod.yaml" target="-blank">/other-cel/enforce-readwriteonce-pod/enforce-readwriteonce-pod.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: readwriteonce-pod
  annotations:
    policies.kyverno.io/title: Enforce ReadWriteOncePod in CEL expressions
    policies.kyverno.io/category: Sample in CEL 
    policies.kyverno.io/subject: PersistentVolumeClaim
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.27-1.28"
    policies.kyverno.io/description: >-
      Some stateful workloads with multiple replicas only allow a single Pod to write
      to a given volume at a time. Beginning in Kubernetes 1.22 and enabled by default
      in 1.27, a new setting called ReadWriteOncePod, available
      for CSI volumes only, allows volumes to be writable from only a single Pod. For more
      information see the blog https://kubernetes.io/blog/2023/04/20/read-write-once-pod-access-mode-beta/.
      This policy enforces that the accessModes for a PersistentVolumeClaim be set to ReadWriteOncePod.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: readwrite-pvc-single-pod
    match:
      any:
      - resources:
          kinds:
          - PersistentVolumeClaim
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        expressions:
          - expression: "'ReadWriteOncePod' in object.spec.accessModes"
            message: "The accessMode must be set to ReadWriteOncePod."


```
