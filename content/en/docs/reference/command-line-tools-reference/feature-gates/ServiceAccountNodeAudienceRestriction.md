---
title: ServiceAccountNodeAudienceRestriction
content_type: feature_gate
_build:
  list: never
  render: false

stages:
  - stage: beta
    defaultValue: true
    fromVersion: "1.32"
---
This gate is used to restrict the audience for which the kubelet can request a service account token for.

