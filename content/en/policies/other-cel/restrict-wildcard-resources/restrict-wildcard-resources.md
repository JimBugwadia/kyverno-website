---
title: "Restrict Wildcards in Resources in CEL expressions"
category: Security, EKS Best Practices in CEL
version: 1.11.0
subject: ClusterRole, Role, RBAC
policyType: "validate"
description: >
    Wildcards ('*') in resources grants access to all of the resources referenced by the given API group and does not follow the principal of least privilege. As much as possible, avoid such open resources unless scoped to perhaps a custom API group. This policy blocks any Role or ClusterRole that contains a wildcard entry in the resources list found in any rule.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/restrict-wildcard-resources/restrict-wildcard-resources.yaml" target="-blank">/other-cel/restrict-wildcard-resources/restrict-wildcard-resources.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-wildcard-resources
  annotations:
    policies.kyverno.io/title: Restrict Wildcards in Resources in CEL expressions
    policies.kyverno.io/category: Security, EKS Best Practices in CEL 
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: ClusterRole, Role, RBAC
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      Wildcards ('*') in resources grants access to all of the resources referenced by
      the given API group and does not follow the principal of least privilege. As much as possible,
      avoid such open resources unless scoped to perhaps a custom API group.
      This policy blocks any Role or ClusterRole that contains a wildcard entry in
      the resources list found in any rule.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: wildcard-resources
      match:
        any:
        - resources:
            kinds:
              - Role
              - ClusterRole
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
            - expression: "object.rules == null || !object.rules.exists(rule, '*' in rule.resources)"
              message: "Use of a wildcard ('*') in any resources is forbidden."


```
