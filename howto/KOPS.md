# KOPS

# Overview

This space is for notes, gotchas and other important information about
using [KOPS](https://github.com/kubernetes/kops) (Kubernetes OPerationS)
for creating and managing Kubernetes clusters in commercial AWS and
GovCloud AWS.

# KOPS in Govcloud

- There are kops AMIs in GovCloud  
  ![AMIs](../images/KOPS/AMIs.png)
- Can we use pre-existing KOPS AMIs?
- If we have to create our own, what are the requirements the base
  Operating System needs to meet?

## Issues

- KOPS AMIs are based on Debian
  - https://github.com/kubernetes/kops/blob/master/docs/operations/images.md#debian
- https://github.com/kubernetes/kops/issues/2034 (closed due toinactivity)
- https://github.com/kubernetes/kops/issues/6325 (closed due to inactivity)

## Gotchas

- Remember to set AWS\_REGION so the AWS SDK and KOPS knows to use
  govcloud namespaces (e.g. arn:aws-us-gov), Example:
  [iam_builder.go](https://github.com/kubernetes/kops/blob/46ba9ff60561aab633396735823a405c11c9ce50/pkg/model/iam/iam_builder.go#L254-L269)
