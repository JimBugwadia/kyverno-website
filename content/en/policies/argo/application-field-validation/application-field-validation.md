---
title: "Application Field Validation"
category: Argo
version: 1.6.0
subject: Application
policyType: "validate"
description: >
    This policy performs some best practices validation on Application fields. Path or chart must be specified but never both. And destination.name or destination.server must be specified but never both.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//argo/application-field-validation/application-field-validation.yaml" target="-blank">/argo/application-field-validation/application-field-validation.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: application-field-validation
  annotations:
    policies.kyverno.io/title: Application Field Validation
    policies.kyverno.io/category: Argo
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Application
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >-
      This policy performs some best practices validation on Application fields.
      Path or chart must be specified but never both. And destination.name or
      destination.server must be specified but never both.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: source-path-chart
      match:
        any:
        - resources:
            kinds:
            - Application
      validate:
        message: >-
          `spec.source.path` OR `spec.source.chart` should be specified but never both.
        anyPattern:
        - spec:
            source:
              path: '?*'
              X(chart):
        - spec:
            source:
              X(path):
              chart: '?*'
    - name: destination-server-name
      match:
        any:
        - resources:
            kinds:
            - Application
      validate:
        message: >-
          `spec.destination.server` OR `spec.destination.name` should be specified but never both.
        anyPattern:
        - spec:
            destination:
              server: '?*'
              X(name):
        - spec:
            destination:
              X(server):
              name: '?*'

```
