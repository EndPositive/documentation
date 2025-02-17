---
# SPDX-FileCopyrightText: 2021 iteratec GmbH
#
# SPDX-License-Identifier: Apache-2.0

title: templates (Directory)
---

## new-hook.yaml

This file contains the specification of your new hook. Please take a look at [ScanCompletionHook | secureCodeBox](/docs/api/crds/scan-completion-hook) on how to configure your `ScanCompletionHook`

## Example

```yaml
apiVersion: "execution.securecodebox.io/v1"
kind: ScanCompletionHook
metadata:
  name: {{ include "generic-webhook.fullname" . }}
  labels:
    {{- include "generic-webhook.labels" . | nindent 4 }}
spec:
  type: ReadOnly
  image: "{{ .Values.hook.image.repository }}:{{ .Values.hook.image.tag | default .Chart.Version }}"
  ttlSecondsAfterFinished: {{ .Values.hook.ttlSecondsAfterFinished }}
  env:
    - name: WEBHOOK_URL
      value: {{ .Values.webhookUrl | quote }}
```

