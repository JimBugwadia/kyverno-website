---
title: "Enforce AppProject with clusterResourceBlacklist"
category: Argo
version: 1.6.0
subject: AppProject
policyType: "validate"
description: >
    An AppProject may optionally specify clusterResourceBlacklist which is a blacklisted group of cluster resources. This is often a good practice to ensure AppProjects do not allow more access than needed. This policy is a combination of two rules which enforce that all AppProjects specify clusterResourceBlacklist and that their group and kind have wildcards as values.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//argo/appproject-clusterresourceblacklist/appproject-clusterresourceblacklist.yaml" target="-blank">/argo/appproject-clusterresourceblacklist/appproject-clusterresourceblacklist.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: appproject-clusterresourceblacklist
  annotations:
    policies.kyverno.io/title: Enforce AppProject with clusterResourceBlacklist
    policies.kyverno.io/category: Argo
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.6.2
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: AppProject
    policies.kyverno.io/description: >-
      An AppProject may optionally specify clusterResourceBlacklist which is a blacklisted
      group of cluster resources. This is often a good practice to ensure AppProjects do
      not allow more access than needed. This policy is a combination of two rules which
      enforce that all AppProjects specify clusterResourceBlacklist and that their group
      and kind have wildcards as values.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: has-wildcard
      match:
        any:
        - resources:
            kinds:
              - AppProject
      preconditions:
        all:
        - key: "{{ request.operation || 'BACKGROUND' }}"
          operator: AnyIn
          value: ["CREATE", "UPDATE"]
      validate:
        message: "Wildcards must be present in group and kind for clusterResourceBlacklist."
        foreach:
        - list: "request.object.spec.clusterResourceBlacklist"
          deny:
            conditions:
              any:
              - key: "{{ contains(element.group, '*') }}"
                operator: Equals
                value: false
              - key: "{{ contains(element.kind, '*') }}"
                operator: Equals
                value: false
    - name: validate-clusterresourceblacklist
      match:
        any:
        - resources:
            kinds:
              - AppProject
      preconditions:
        all:
        - key: "{{ request.operation || 'BACKGROUND' }}"
          operator: AnyIn
          value: ["CREATE", "UPDATE"]
      validate:
        message: "AppProject must specify clusterResourceBlacklist."
        deny:
          conditions:
            any:
            - key: clusterResourceBlacklist
              operator: AnyNotIn
              value: "{{ request.object.spec.keys(@) }}"

```
