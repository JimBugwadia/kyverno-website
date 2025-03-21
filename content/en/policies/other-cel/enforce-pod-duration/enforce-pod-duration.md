---
title: "Enforce pod duration in CEL expressions"
category: Sample in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    This validation is valuable when annotations are used to define durations, such as to ensure a Pod lifetime annotation does not exceed some site specific max threshold. Pod lifetime annotation can be no greater than 8 hours.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/enforce-pod-duration/enforce-pod-duration.yaml" target="-blank">/other-cel/enforce-pod-duration/enforce-pod-duration.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: pod-lifetime
  annotations:
    policies.kyverno.io/title: Enforce pod duration in CEL expressions
    policies.kyverno.io/category: Sample in CEL 
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      This validation is valuable when annotations are used to define durations,
      such as to ensure a Pod lifetime annotation does not exceed some site specific max threshold.
      Pod lifetime annotation can be no greater than 8 hours.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: pods-lifetime
    match:
      any:
      - resources:
          kinds:
          - Pod
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        variables:
          - name: hasLifetimeAnnotation
            expression: "object.metadata.?annotations[?'pod.kubernetes.io/lifetime'].hasValue()"
          - name: lifetimeAnnotationValue
            expression: "variables.hasLifetimeAnnotation ? object.metadata.annotations['pod.kubernetes.io/lifetime'] : '0s'"
        expressions:
          - expression: "!(duration(variables.lifetimeAnnotationValue) > duration('8h'))"
            message: "Pod lifetime exceeds limit of 8h"


```
