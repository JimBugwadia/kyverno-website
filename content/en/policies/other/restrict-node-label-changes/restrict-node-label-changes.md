---
title: "Restrict node label changes"
category: Sample
version: 1.6.0
subject: Node, Label
policyType: "validate"
description: >
    Node labels are critical pieces of metadata upon which many other applications and logic may depend and should not be altered or removed by regular users. This policy prevents changes or deletions to a label called `foo` on cluster Nodes. Use of this policy requires removal of the Node resource filter in the Kyverno ConfigMap ([Node,*,*]). Due to Kubernetes CVE-2021-25735, this policy requires, at minimum, one of the following versions of Kubernetes: v1.18.18, v1.19.10, v1.20.6, or v1.21.0.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/restrict-node-label-changes/restrict-node-label-changes.yaml" target="-blank">/other/restrict-node-label-changes/restrict-node-label-changes.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: protect-node-label-foo
  annotations:
    policies.kyverno.io/title: Restrict node label changes
    policies.kyverno.io/category: Sample
    policies.kyverno.io/subject: Node, Label
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/description: >-
      Node labels are critical pieces of metadata upon which many other applications and
      logic may depend and should not be altered or removed by regular users.
      This policy prevents changes or deletions to a label called `foo` on
      cluster Nodes. Use of this policy requires removal of the Node resource filter
      in the Kyverno ConfigMap ([Node,*,*]). Due to Kubernetes CVE-2021-25735, this policy
      requires, at minimum, one of the following versions of Kubernetes:
      v1.18.18, v1.19.10, v1.20.6, or v1.21.0.
spec:
  validationFailureAction: Enforce
  background: false
  rules:
  - name: prevent-label-value-changes
    match:
      any:
      - resources:
          kinds:
          - Node
    validate:
      allowExistingViolations: false
      message: "Modifying the `foo` label on a Node is not allowed."
      deny:
        conditions:
          all:
          - key: "{{ request.object.metadata.labels.foo || '' }}"
            operator: NotEquals
            value: ""
          - key: "{{ request.object.metadata.labels.foo || '' }}"
            operator: NotEquals
            value: "{{ request.oldObject.metadata.labels.foo || '' }}"
  - name: prevent-label-key-removal
    match:
      any:
      - resources:
          kinds:
          - Node
    preconditions:
      all:
      - key: "{{ request.operation || 'BACKGROUND' }}"
        operator: Equals
        value: UPDATE
      - key: "{{ request.oldObject.metadata.labels.foo || '' }}"
        operator: Equals
        value: "?*"
    validate:
      allowExistingViolations: false
      message: "Removing the `foo` label on a Node is not allowed."
      pattern:
        metadata:
          labels:
            foo: "*"

```
