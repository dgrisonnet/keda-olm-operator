<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [KEDA OLM Operator](#keda-olm-operator)
  - [Installation](#installation)
    - [Operator Hub Installation](#operator-hub-installation)
    - [Manual installation](#manual-installation)
  - [The `KedaController` Custom Resource](#the-kedacontroller-custom-resource)
    - [`KedaController` Spec](#kedacontroller-spec)
  - [Uninstallation](#uninstallation)
    - [How to uninstall KEDA Controller](#how-to-uninstall-keda-controller)
    - [How to uninstall KEDA OLM Operator](#how-to-uninstall-keda-olm-operator)
  - [Development](#development)
    - [Operator Framework](#operator-framework)
    - [Running locally](#running-locally)
    - [Building the Operator Image](#building-the-operator-image)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# KEDA OLM Operator

<p style="font-size: 25px" align="center">
<a href="https://github.com/kedacore/keda-olm-operator/actions"><img src="https://github.com/kedacore/keda-olm-operator/workflows/main%20build/badge.svg" alt="main build"></a>
<a href="https://github.com/kedacore/keda-olm-operator/actions"><img src="https://github.com/kedacore/keda-olm-operator/workflows/nightly%20tests/badge.svg" alt="nightly e2e"></a></p>


Operator for deploying KEDA controller on OpenShift or any Kubernetes cluster with
[Operator Lifecycle Manager](https://github.com/operator-framework/operator-lifecycle-manager) framework installed.

## Installation

Please note that you can not run both KEDA v1 and v2 on the same Kubernetes cluster. You need to uninstall KEDA v1 first, in order to install and use KEDA v2.
Don't forget to uninstall KEDA v1 CRDs as well, to ensure that, please run:
```bash
kubectl delete crd scaledobjects.keda.k8s.io
kubectl delete crd triggerauthentications.keda.k8s.io

```


### Operator Hub Installation
1. On Operator Hub Marketplace locate and install `KEDA` operator
2. Create `KedaController` resource in `keda` namespace

![Operator Hub Installation Demo](images/keda-olm-install.gif)


### Manual installation

The following will install [KEDA](https://github.com/kedacore/keda) and configure it
appropriately for your cluster, please run these commands:

```bash
make deploy                                                                  # deploy KEDA OLM Operator
kubectl apply -n keda -f config/samples/keda_v1alpha1_kedacontroller.yaml    # install KEDA
```

To be clear, the operator will be deployed in the `keda` namespace,
and then it will install KEDA into this namespace.

## The `KedaController` Custom Resource

The installation of KEDA is triggered by the creation of
[a `KedaController` custom resource](config/samples/keda_v1alpha1_kedacontroller.yaml).
Only custom resource named `keda` in namespace `keda` will trigger the installation,
reconfiguration, or removal of the KEDA Controller resources.

There could be only one KEDA Controller in the cluster.

### `KedaController` Spec
```
apiVersion: keda.sh/v1alpha1
kind: KedaController
metadata:
  name: keda
  namespace: keda
spec:
  ###
  # THERE SHOULD BE ONLY ONE INSTANCE OF THIS RESOURCE PER CLUSTER
  # with Name set to 'keda' created in namespace 'keda'
  ###

  ## Namespace that should be watched by KEDA Controller,
  # omit or set empty to watch all namespaces (default setting)
  watchNamespace: ""

  ## Logging level for KEDA Operator
  # allowed values: 'debug', 'info', 'error', or an integer value greater than 0, specified as string
  # default value: info
  logLevel: info

  ## Logging format for KEDA Operator
  # allowed values are json and console
  # default value: console
  logEncoder: console

  ## Logging level for Metrics Server
  # allowed values: "0" for info, "4" for debug, or an integer value greater than 0, specified as string
  # default value: "0"
  logLevelMetrics: "0"

  ## Node selector for pod scheduling - both KEDA Operator and Metrics Server
  # https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
  # nodeSelector:
  #  beta.kubernetes.io/os: linux

  ## Tolerations for pod scheduling - both KEDA Operator and Metrics Server
  # https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
  # tolerations:
  # - key: "key1"
  #   operator: "Equal"
  #   value: "value1"
  #   effect: "NoSchedule"
  # - key: "key1"
  #   operator: "Equal"
  #   value: "value1"
  #   effect: "NoExecute"

  ## Affinity for pod scheduling - both KEDA Operator and Metrics Server
  # https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/
  # affinity:
  #  podAntiAffinity:
  #    requiredDuringSchedulingIgnoredDuringExecution:
  #     - labelSelector:
  #         matchExpressions:
  #         - key: app
  #           operator: In
  #           values:
  #           - keda-operator
  #           - keda-operator-metrics-apiserver
  #       topologyKey: "kubernetes.io/hostname"

  ## Pod priority for KEDA Operator and Metrics Adapter
  # https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/
  # priorityClassName: high-priority

  ## Manage resource requests & limits for KEDA Operator
  # https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
  # resourcesKedaOperator:
  #   limits:
  #     cpu: 100m
  #     memory: 100Mi
  #   requests:
  #     cpu: 1000m
  #     memory: 1000Mi

  ## Manage resource requests & limits for KEDA Metrics Server
  # https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
  # resourcesMetricsServer:
  #   limits:
  #     cpu: 100m
  #     memory: 100Mi
  #   requests:
  #     cpu: 1000m
  #     memory: 1000Mi
```


## Uninstallation

### How to uninstall KEDA Controller
Locate installed `KEDA` Operator in `keda` namespace and then remove created `KedaController` resoure or simply delete the `KedaController` resource:

```bash
kubectl delete -n keda -f config/samples/keda_v1alpha1_kedacontroller.yaml
```

### How to uninstall KEDA OLM Operator
To remove KEDA OLM Operator from your cluster, on Operator Hub locate and uninstall `KEDA` operator.

In case of manual installation, run these commands:

```bash
make undeploy
```

## Development

### Operator Framework

This operator was created using the
[operator-sdk](https://github.com/operator-framework/operator-sdk/). And uses
[Operator Lifecycle
Manager](https://github.com/operator-framework/operator-lifecycle-manager)
to describe deployment metadata.

### Running locally
It can be convenient to run the operator outside of the cluster to
test changes. The following command will build the operator and use
your current "kube config" to connect to the cluster:

```bash
make install    # install KedaController CRD in the cluster
make run        # run operator locally
```

### Building the Operator Image

To build the operator:

```bash
make build
```
