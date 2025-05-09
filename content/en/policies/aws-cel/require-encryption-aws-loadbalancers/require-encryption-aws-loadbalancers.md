---
title: "Require Encryption with AWS LoadBalancers in CEL expressions"
category: AWS, EKS Best Practices in CEL
version: 
subject: Service
policyType: "validate"
description: >
    Services of type LoadBalancer when deployed inside AWS have support for transport encryption if it is enabled via an annotation. This policy requires that Services of type LoadBalancer contain the annotation service.beta.kubernetes.io/aws-load-balancer-ssl-cert with some value.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//aws-cel/require-encryption-aws-loadbalancers/require-encryption-aws-loadbalancers.yaml" target="-blank">/aws-cel/require-encryption-aws-loadbalancers/require-encryption-aws-loadbalancers.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-encryption-aws-loadbalancers
  annotations:
    policies.kyverno.io/title: Require Encryption with AWS LoadBalancers in CEL expressions
    policies.kyverno.io/category: AWS, EKS Best Practices in CEL 
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Service
    kyverno.io/kyverno-version: 1.12.1
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      Services of type LoadBalancer when deployed inside AWS have support for
      transport encryption if it is enabled via an annotation. This policy requires
      that Services of type LoadBalancer contain the annotation
      service.beta.kubernetes.io/aws-load-balancer-ssl-cert with some value.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: aws-loadbalancer-has-ssl-cert
    match:
      any:
      - resources:
          kinds:
          - Service
          operations:
          - CREATE
          - UPDATE
    celPreconditions: 
      - name: "type-should-be-load-balancer"
        expression: "object.spec.type == 'LoadBalancer'"
    validate:
      cel:
        expressions:
          - expression: >-
              object.metadata.?annotations[?'service.beta.kubernetes.io/aws-load-balancer-ssl-cert'].orValue('') != ''
            message: "Service of type LoadBalancer must carry the annotation service.beta.kubernetes.io/aws-load-balancer-ssl-cert."


```
