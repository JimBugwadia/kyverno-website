---
title: "Restrict Secrets by Name"
category: Other
version: 
subject: Pod, Secret
policyType: "validate"
description: >
    Secrets often contain sensitive information and their access should be carefully controlled. Although Kubernetes RBAC can be effective at restricting them in several ways, it lacks the ability to use wildcards in resource names. This policy ensures that only Secrets beginning with the name `safe-` can be consumed by Pods. In order to work effectively, this policy needs to be paired with a separate policy or rule to require `automountServiceAccountToken=false` since this would otherwise result in a Secret being mounted.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/restrict-secrets-by-name/restrict-secrets-by-name.yaml" target="-blank">/other/restrict-secrets-by-name/restrict-secrets-by-name.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-secrets-by-name
  annotations:
    policies.kyverno.io/title: Restrict Secrets by Name
    policies.kyverno.io/category: Other
    policies.kyverno.io/subject: Pod, Secret
    kyverno.io/kyverno-version: 1.6.0
    kyverno.io/kubernetes-version: "1.21"
    policies.kyverno.io/description: >-
      Secrets often contain sensitive information and their access should be carefully controlled.
      Although Kubernetes RBAC can be effective at restricting them in several ways,
      it lacks the ability to use wildcards in resource names. This policy ensures
      that only Secrets beginning with the name `safe-` can be consumed by Pods.
      In order to work effectively, this policy needs to be paired with a separate policy
      or rule to require `automountServiceAccountToken=false` since this would otherwise
      result in a Secret being mounted.
spec:
  background: false
  validationFailureAction: Enforce
  rules:
  - name: safe-secrets-from-env
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
        - CREATE
        - UPDATE
    validate:
      message: "Only Secrets beginning with `safe-` may be consumed in env statements."
      pattern:
        spec:
          =(ephemeralContainers):
          - =(name): "*"
            =(env):
            - =(valueFrom):
                =(secretKeyRef):
                    name: safe-*
          =(initContainers):
          - =(name): "*"
            =(env):
            - =(valueFrom):
                =(secretKeyRef):
                    name: safe-*
          containers:
          - name: "*"
            =(env):
            - =(valueFrom):
                =(secretKeyRef):
                    name: safe-*
  - name: safe-secrets-from-envfrom
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
        - CREATE
        - UPDATE
    validate:
      message: "Only Secrets beginning with `safe-` may be consumed in envFrom statements."
      pattern:
        spec:
          =(ephemeralContainers):
          - =(name): "*"
            =(envFrom):
            - =(secretRef):
                name: safe-*
          =(initContainers):
          - =(name): "*"
            =(envFrom):
            - =(secretRef):
                name: safe-*
          containers:
          - name: "*"
            =(envFrom):
            - =(secretRef):
                name: safe-*
  - name: safe-secrets-from-volumes
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
        - CREATE
        - UPDATE
    validate:
      message: "Only Secrets beginning with `safe-` may be consumed in volumes."
      pattern:
        spec:
          =(volumes):
          - =(secret):
              secretName: safe-*
```
