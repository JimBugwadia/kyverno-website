---
title: "Limit hostPath Volumes to Specific Directories"
category: Other
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    hostPath volumes consume the underlying node's file system. If hostPath volumes are not to be universally disabled, they should be restricted to only certain host paths so as not to allow access to sensitive information. This policy ensures the only directory that can be mounted as a hostPath volume is /data. It is strongly recommended to pair this policy with a second to ensure readOnly access is enforced preventing directory escape.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/limit-hostpath-vols/limit-hostpath-vols.yaml" target="-blank">/other/limit-hostpath-vols/limit-hostpath-vols.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: limit-hostpath-vols
  annotations:
    policies.kyverno.io/title: Limit hostPath Volumes to Specific Directories
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kyverno-version: 1.6.2
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      hostPath volumes consume the underlying node's file system. If hostPath volumes
      are not to be universally disabled, they should be restricted to only certain
      host paths so as not to allow access to sensitive information. This policy ensures
      the only directory that can be mounted as a hostPath volume is /data. It is strongly
      recommended to pair this policy with a second to ensure readOnly
      access is enforced preventing directory escape.
spec:
  background: false
  validationFailureAction: Audit
  rules:
  - name: limit-hostpath-to-slash-data
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{ request.object.spec.volumes[?hostPath] | length(@) }}"
        operator: GreaterThanOrEquals
        value: 1
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
        - CREATE
        - UPDATE
    validate:
      message: hostPath volumes are confined to /data.
      foreach:
      - list: "request.object.spec.volumes[?hostPath].hostPath"
        deny:
          conditions:
            any:
            - key: "{{ element.path  | to_string(@) | split(@, '/') | [1] }}"
              operator: NotEquals
              value: data
```
