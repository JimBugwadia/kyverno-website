---
title: "Require Ingress HTTPS"
category: Other
version: 
subject: Ingress
policyType: "validate"
description: >
    Ingress resources should only allow secure traffic by disabling HTTP and therefore only allowing HTTPS. This policy requires that all Ingress resources set the annotation `kubernetes.io/ingress.allow-http` to `"false"` and specify TLS in the spec.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/require-ingress-https/require-ingress-https.yaml" target="-blank">/other/require-ingress-https/require-ingress-https.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-ingress-https
  annotations:
    policies.kyverno.io/title: Require Ingress HTTPS
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.9.0
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/subject: Ingress
    policies.kyverno.io/description: >-
      Ingress resources should only allow secure traffic by disabling
      HTTP and therefore only allowing HTTPS. This policy requires that all
      Ingress resources set the annotation `kubernetes.io/ingress.allow-http` to
      `"false"` and specify TLS in the spec.
spec:
  background: true
  validationFailureAction: Audit
  rules:
  - name: has-annotation
    match:
      any:
      - resources:
          kinds:
          - Ingress
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
        - CREATE
        - UPDATE
    validate:
      message: "The kubernetes.io/ingress.allow-http annotation must be set to false."
      pattern:
        metadata:
          annotations:
            kubernetes.io/ingress.allow-http: "false"
  - name: has-tls
    match:
      any:
      - resources:
          kinds:
          - Ingress
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
        - CREATE
        - UPDATE
    validate:
      message: "TLS must be defined."
      deny:
        conditions:
          all:
          - key: tls
            operator: AnyNotIn
            value: "{{ request.object.spec.keys(@) }}"

```
