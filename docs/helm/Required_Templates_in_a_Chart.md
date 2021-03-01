# Required filed in Helm Chart Templates

All `.yaml` files will be sent over to Kubernetes on installation of the chart.
If you created the chart with `helm create <name>` helm will pre-populate the
templates directory with some examples. We're going over these examples here.

## service.yaml

```go-template
apiVersion: v1
kind: Service
metadata:
  name: {{ include "test.fullname" . }}
  labels:
    {{- include "test.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "test.selectorLabels" . | nindent 4 }}
```

- The prefix `test` comes from the chart's name which is also `test` here
- The `include` commands include definitions from the `_helpers.tpl` file.
- `.Values` comes from `values.yaml` and can be overridden by the user when
  installing the chart.

This file describes a Kubernetes service which can be replicated.
See the [Service Documentation](https://kubernetes.io/docs/reference/kubernetes-api/services-resources/service-v1/)
for more information.

## serviceaccount.yaml

```go-template
{{- if .Values.serviceAccount.create -}}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "test.serviceAccountName" . }}
  labels:
    {{- include "test.labels" . | nindent 4 }}
  {{- with .Values.serviceAccount.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
{{- end }}
```

This file describes a service account. This is completely optional.

Find the documentation for [Service Accounts](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/service-account-v1/)
on the Kubernetes site.

## ingress.yaml

```go-template
{{- if .Values.ingress.enabled -}}
{{- $fullName := include "test.fullname" . -}}
{{- $svcPort := .Values.service.port -}}
{{- if semverCompare ">=1.14-0" .Capabilities.KubeVersion.GitVersion -}}
apiVersion: networking.k8s.io/v1beta1
{{- else -}}
apiVersion: extensions/v1beta1
{{- end }}
kind: Ingress
metadata:
  name: {{ $fullName }}
  labels:
    {{- include "test.labels" . | nindent 4 }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  {{- if .Values.ingress.tls }}
  tls:
    {{- range .Values.ingress.tls }}
    - hosts:
        {{- range .hosts }}
        - {{ . | quote }}
        {{- end }}
      secretName: {{ .secretName }}
    {{- end }}
  {{- end }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            backend:
              serviceName: {{ $fullName }}
              servicePort: {{ $svcPort }}
          {{- end }}
    {{- end }}
  {{- end }}
```

This describes the ingress-point of a service, as you can see Helm feature-gates
this completely (by `.Values.ingress.enabled`). If you do not need routing but
do it yourself you can skip this file but you will gain flexibility by including
it. This definition integrates with a defined ingress deployment
(e.g. `ingress-nginx`) to extend its definition, so if you ever want to run
your service with an ingress you have to include this file to be able to define
granular routes. If you don't you only can route complete parts of an URL tree
to the service (e.g. everything below `/bla/fasel` goes to the service, reverse
proxy style)

Find the [Ingress Documentation](https://kubernetes.io/docs/reference/kubernetes-api/services-resources/ingress-v1/)
on the Kubernetes site for more information.

## deployment.yaml

```go-template
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "test.fullname" . }}
  labels:
    {{- include "test.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "test.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      labels:
        {{- include "test.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "test.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

This defines a deployment of your defined service. It absolutely includes all
possibilities of what Kubernetes can do, so better strip it down to a manageable
size before committing it to your SCM.

The most interesting part of the Kubernetes documentation is the [Documentation of PodSpec](https://kubernetes.io/docs/reference/kubernetes-api/workloads-resources/pod-v1/#PodSpec)
for this file (e.g. everything below `spec.template.spec`).

## hpa.yaml

```go-template
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "test.fullname" . }}
  labels:
    {{- include "test.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "test.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
    {{- if .Values.autoscaling.targetCPUUtilizationPercentage }}
    - type: Resource
      resource:
        name: cpu
        targetAverageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
    {{- end }}
    {{- if .Values.autoscaling.targetMemoryUtilizationPercentage }}
    - type: Resource
      resource:
        name: memory
        targetAverageUtilization: {{ .Values.autoscaling.targetMemoryUtilizationPercentage }}
    {{- end }}
{{- end }}
```

This is the definition file for the Horizontal Pod Autoscaler. You can leave it
as is to be able to autoscale over RAM and CPU consumption, but be aware this
may be not the best option to scale.

You can find the [Horizontal Pod Autoscaler Documentation](https://kubernetes.io/docs/reference/kubernetes-api/workloads-resources/horizontal-pod-autoscaler-v2beta2/)
on the Kubernetes site to get more information.

## _helpers.tpl

This file will not be part of the Kubernetes manifest file as it starts with an
underscore.

```go-template
{{/*
Expand the name of the chart.
*/}}
{{- define "test.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
If release name contains chart name it will be used as a full name.
*/}}
{{- define "test.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "test.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "test.labels" -}}
helm.sh/chart: {{ include "test.chart" . }}
{{ include "test.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "test.selectorLabels" -}}
app.kubernetes.io/name: {{ include "test.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Create the name of the service account to use
*/}}
{{- define "test.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "test.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}

```