# Kubernetes basic  monitoring

![2.21.2](https://img.shields.io/badge/Datadog%20chart-2.21.2-632ca6?labelColor=f0f0f0&logo=Helm&logoColor=0f1689)
![7.30.0](https://img.shields.io/badge/Agent-7.30.0-632ca6?&labelColor=f0f0f0&logo=Datadog&logoColor=632ca6)
![1.14.0](https://img.shields.io/badge/Cluster%20Agent-1.14.0-632ca6?labelColor=f0f0f0&logo=Datadog&logoColor=632ca6)
![1.22.0](https://img.shields.io/badge/Kubernetes-1.22.0-326ce5?labelColor=f0f0f0&logo=Kubernetes&logoColor=326ce5)

## Install

Install [Datadog Helm Chart](https://github.com/DataDog/helm-charts/tree/master/charts/datadog)

```bash
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

Create a namespace for Datadog

```bash
kubectl create ns datadog
```

Create a secret with the [API key](https://app.datadoghq.com/account/settings#api).

```bash
kubectl create secret generic datadog-keys -n datadog --from-literal=api-key=<API-KEY>
```

Create the `values.yaml` file.

```yaml
# datadog-values.yaml
# Archivo de valores de Helm actualizado para desplegar el agente de Datadog en Kubernetes (Linux only)

## Clave API (requerida)
datadog:
  apiKeyExistingSecret: datadog-secret
  site: datadoghq.eu  # Cambia según tu región
  clusterName: my-k8s-cluster  # Cambia según tu entorno

  ## Etiquetas unificadas (recomendadas)
  tags:
    - env:production
    - team:infra
    - service:k8s-monitoring

  ## Logs (opcional)
  logs:
    enabled: true
    containerCollectAll: true

  ## APM (opcional)
  apm:
    portEnabled: false  # true si quieres abrir puerto 8126 para traces

  ## Live Containers / Orchestrator Explorer
  processAgent:
    enabled: true
  orchestratorExplorer:
    enabled: true

  ## Autodiscovery y Cluster Checks
  clusterChecksEnabled: true
  kubeStateMetricsCore:
    enabled: true

## Cluster Agent (recomendado)
clusterAgent:
  enabled: true


## Recursos opcionales (ajustables)
agents:
  tolerations:
    - key: "node-role.kubernetes.io/master"
      operator: "Exists"
      effect: "NoSchedule"
  resources:
    limits:
      memory: 512Mi
    requests:
      cpu: 200m
      memory: 256Mi

```

All yaml snippets presented from here are expected to be **propertly merged** into the main `values.yaml`.

Check distribution specific notes.  

- [kubeadm/vanilla](kubeadm.md)
- [AKS](aks.md)
- [EKS](eks.md)
- [Red Hat OpenShift Container Platform v4](openshift4.md)

 Deploy with the command below.

```bash
helm install datadog datadog/datadog -n datadog -f values.yaml
```

Complete values file examples can be found [here](examples).

**Note**: OpenShift [metrics](https://docs.datadoghq.com/integrations/openshift/#metrics) are all about quotas.  The statment below must return something for OpenShift specific metrics to show up.

`oc get clusterresourcequotas --all-namespaces`
