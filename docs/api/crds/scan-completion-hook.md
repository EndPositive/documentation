---
# SPDX-FileCopyrightText: 2021 iteratec GmbH
#
# SPDX-License-Identifier: Apache-2.0

title: "ScanCompletionHook"
---

ScanCompletionHooks are Custom Resource Definitions (CRD's) used to define custom behavior which should be run after a scan has been completed.

For more detailed explanations on how a new hook can be integrated, see the [hooks section](/docs/contributing/integrating-a-hook) in the contribution part of our docs.

## Specification (Spec)

### Type (Required)

The `type` field can be either `ReadOnly` or `ReadAndWrite`.

`ReadOnly` hooks only have read rights on the findings and the raw scan reports (e.g. XML output from nmap). These are usually used to export the findings into a external system like "Elasticsearch" or "DefectDojo" or to send out notifications to chats like "Slack". ReadOnly hooks are executed in parallel to speed up their runtime.

`ReadAndWrite` hooks have the ability to update both the findings and raw scan reports. This can be used to attach additional metadata to the findings by comparing the findings to external inventory systems or APIs of cloud providers.

### Image (Required)

The `image` field contains a container image reference for the image supposed to run as the hook.

### ImagePullSecrets (Optional)

The `imagePullSecrets` field can be used to specify pull secrets used to access the hooks image from a private registry.

### ImagePullPolicy (Optional)
The `imagePullPolicy` field can be used to specify under which circumstances the images should be pulled from the registry.
One of Always, Never, IfNotPresent. Defaults to Always if :latest tag is specified, or IfNotPresent otherwise.
See the [Kubernetes docs](https://kubernetes.io/docs/concepts/containers/images#updating-images) for more information.
	
### Env (Optional)

The `env` field can be used to specify env variables and to mount secrets into containers.

### Volumes (Optional)

`volumes` lets you specify Kubernetes volumes that you want to use and make available to the hook.
Similarly to `env`, it can be used to pass data into a container.
It has to be combined with [`volumeMounts`](#volumemounts-optional) to be useful (see below).

### VolumeMounts (Optional)

`volumeMounts` let you specify where you want the previously-created volumes to be mounted inside the container.
It has the same API as the `volumeMounts` property on Kubernetes pods.

### ServiceAccountName (Optional)

The `serviceAccountName` field can be used to specify a custom ServiceAccount to use for the Kubernetes Job running the hook.
Should only be used if your hook needs specific RBAC Access. Otherwise the hook is run using a `scan-completion-hook` service account.

The service account should have at least `get` rights on `scans.execution.securecodebox.io`, and `get` & `patch` on `scans.execution.securecodebox.io/status` so that the hooks can work correctly.

### TTLSecondsAfterFinished (Optional)

`ttlSecondsAfterFinished` can be used to automatically delete the completed Kubernetes job used to run the hook.
This sets the `ttlSecondsAfterFinished` field on the created job. This requires your cluster to have the [TTLAfterFinished](https://kubernetes.io/docs/concepts/workloads/controllers/ttlafterfinished/) feature gate enabled in your cluster.

## Example

```yaml
apiVersion: execution.securecodebox.io/v1
kind: ScanCompletionHook
metadata:
  name: elastic-persistence-hook
spec:
  type: ReadOnly
  image: docker.io/securecodebox/persistence-elastic:latest
  imagePullSecrets:
  - name: image-pull-secret
  serviceAccountName: elastic-persistence
  env:
  - name: ELASTICSEARCH_ADDRESS
    value: https://data.chase.securecodebox.io
  - name: ELASTICSEARCH_USERNAME
    valueFrom:
      secretKeyRef:
        key: username
        name: elastic-persistence-credentials
  - name: ELASTICSEARCH_PASSWORD
    valueFrom:
      secretKeyRef:
        key: password
        name: elastic-persistence-credentials
  ttlSecondsAfterFinished: 60
```
