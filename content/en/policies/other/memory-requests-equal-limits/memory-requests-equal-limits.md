---
title: "Memory Requests Equal Limits"
category: Sample
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    Pods which have memory limits equal to requests could be given a QoS class of Guaranteed if they also set CPU limits equal to requests. Guaranteed is the highest schedulable class.  This policy checks that all containers in a given Pod have memory requests equal to limits.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/memory-requests-equal-limits/memory-requests-equal-limits.yaml" target="-blank">/other/memory-requests-equal-limits/memory-requests-equal-limits.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: memory-requests-equal-limits
  annotations:
    policies.kyverno.io/title: Memory Requests Equal Limits
    policies.kyverno.io/category: Sample
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/description: >-
      Pods which have memory limits equal to requests could be given a QoS class of Guaranteed if
      they also set CPU limits equal to requests. Guaranteed is the highest schedulable class. 
      This policy checks that all containers in a given Pod have memory requests equal to limits.
spec:
  validationFailureAction: Audit
  background: false
  rules:
  - name: memory-requests-equal-limits
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "resources.requests.memory must be equal to resources.limits.memory"
      deny:
        conditions:
          any:
          - key: "{{ request.object.spec.containers[?resources.requests.memory!=resources.limits.memory] | length(@) }}"
            operator: NotEquals
            value: 0

```
