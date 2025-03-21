---
title: "Block Velero Restore to Protected Namespace"
category: Velero
version: 
subject: Restore
policyType: "validate"
description: >
    Velero allows on backup and restore operations and is designed to be run with full cluster admin permissions. It allows on cross namespace restore operations, which means you can restore backup of namespace A to namespace B. This policy protect restore operation into system or any protected namespaces, listed in deny condition section.  It checks the Restore CRD object and its namespaceMapping field. If destination match protected namespace then operation fails and warning message is throw.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//velero/block-velero-restore/block-velero-restore.yaml" target="-blank">/velero/block-velero-restore/block-velero-restore.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: block-velero-restore
  annotations:
    policies.kyverno.io/title: Block Velero Restore to Protected Namespace
    policies.kyverno.io/category: Velero
    policies.kyverno.io/subject: Restore
    policies.kyverno.io/description: >-
      Velero allows on backup and restore operations and is designed to be run with full cluster admin permissions.
      It allows on cross namespace restore operations, which means you can restore backup of namespace A to namespace B.
      This policy protect restore operation into system or any protected namespaces, listed in deny condition section. 
      It checks the Restore CRD object and its namespaceMapping field. If destination match protected namespace
      then operation fails and warning message is throw.
spec:
  validationFailureAction: Audit
  background: false
  rules:
  - name: block-velero-restore-to-protected-namespace
    match:
      any:
      - resources:
          kinds:
          - velero.io/v1/Restore
    validate:
      message: "Warning! Restore to protected namespace: {{request.object.spec.namespaceMapping | values(@)}} is not allowed!"
      deny:
        conditions:
          any:
            - key: "{{request.object.spec.namespaceMapping || `{}` | values(@)}}"
              operator: AnyIn
              value:
              - kube-system
              - kube-node-lease

```
