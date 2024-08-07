---
title: "Restrict Jobs in CEL expressions"
category: Other in CEL
version: 
subject: Job
policyType: "validate"
description: >
    Jobs can be created directly and indirectly via a CronJob controller. In some cases, users may want to only allow Jobs if they are created via a CronJob. This policy restricts Jobs so they may only be created by a CronJob.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/restrict-jobs/restrict-jobs.yaml" target="-blank">/other-cel/restrict-jobs/restrict-jobs.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-jobs
  annotations:
    policies.kyverno.io/title: Restrict Jobs in CEL expressions
    policies.kyverno.io/category: Other in CEL 
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Job
    kyverno.io/kyverno-version: 1.12.1
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      Jobs can be created directly and indirectly via a CronJob controller.
      In some cases, users may want to only allow Jobs if they are created via a CronJob.
      This policy restricts Jobs so they may only be created by a CronJob.
spec:
  validationFailureAction: Enforce
  rules:
    - name: restrict-job-from-cronjob
      match:
        any:
        - resources:
            kinds:
              - Job
      celPreconditions:
        - name: "not-created-by-cronjob"
          expression: "!has(object.metadata.ownerReferences) || object.metadata.ownerReferences[0].kind != 'CronJob'"
      validate:
        cel:
          expressions:
            - expression: "false"
              message: Jobs are only allowed if spawned from CronJobs.


```
