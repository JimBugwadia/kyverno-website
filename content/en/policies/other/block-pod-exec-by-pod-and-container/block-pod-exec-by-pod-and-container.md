---
title: "Block Pod Exec by Pod and Container"
category: Sample
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    The `exec` command may be used to gain shell access, or run other commands, in a Pod's container. While this can be useful for troubleshooting purposes, it could represent an attack vector and is discouraged. This policy blocks Pod exec commands to containers named `nginx` in Pods starting with name `myapp-maintenance`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/block-pod-exec-by-pod-and-container/block-pod-exec-by-pod-and-container.yaml" target="-blank">/other/block-pod-exec-by-pod-and-container/block-pod-exec-by-pod-and-container.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: deny-exec-by-pod-and-container
  annotations:
    policies.kyverno.io/title: Block Pod Exec by Pod and Container
    policies.kyverno.io/category: Sample
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      The `exec` command may be used to gain shell access, or run other commands, in a Pod's container. While this can
      be useful for troubleshooting purposes, it could represent an attack vector and is discouraged.
      This policy blocks Pod exec commands to containers named `nginx` in Pods starting
      with name `myapp-maintenance`.
spec:
  validationFailureAction: Enforce
  background: false
  rules:
  - name: deny-nginx-exec-in-myapp-maintenance
    match:
      any:
      - resources:
          kinds:
          - Pod/exec
    preconditions:
      all:
      - key: "{{ request.operation || 'BACKGROUND' }}"
        operator: Equals
        value: CONNECT
      - key: "{{ request.name }}"
        operator: Equals
        value: myapp-maintenance*
    validate:
      message: Nginx containers inside myapp-maintanence Pods may not be exec'd into.
      deny:
        conditions:
          all:
          - key: "{{ request.object.container }}"
            operator: Equals
            value: nginx

```
