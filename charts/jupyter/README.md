# jupyter

Start a Jupyter Notebook running on a PyTorch image from Nvidia on DGX.
As standard create a disk with x"100Gi" mounted in /data and using one nvidia GPU.

## TL;DR

```console
helm repo add nvidia-charts https://ki-smile.github.io/nvidia-charts
helm repo update
helm install jupyter nvidia-charts/jupyter
```

## Add Helm repo

Before we can install charts from a repo we need to add it and fetch the information:

```console
helm repo add nvidia-charts https://ki-smile.github.io/nvidia-charts
helm repo update
```

## Installing the Chart

To install the chart with the release name `rel-jupyter`. The idea with the Helm release concept is that you can have multiple installations of the charts in the same namespace.

```console
helm install rel-jupyter nvidia-charts/jupyter
```

When have started the installation you get futher information about how to connect to the instance.

## Scale down

When you don't need the resources (the nvidia-gpu's is a limited resource in the cluster), scale down the deployment so it's possible for other to use it.

```console
kubectl scale deployment rel-jupyter --replicas=0
```

## Scale up

When you want to continue the work on a scaled down release you can scale up the deployment again

```console
kubectl scale deployment rel-jupyter --replicas=1
```

## Uninstalling the Chart

To uninstall the `rel-jupyter` release/deployment and remove all resources (including the data)

```console
helm uninstall alertmanager-bot
```

The command removes all the Kubernetes components associated with the chart **including persistent volumes** and deletes the release.

## Configuration

Read through the [values.yaml](./values.yaml)

If you want to override those values, you can put them in a yaml files, see [2gpu.yaml](./2gpu.yaml) for a sample using two gpu's instead of one.

```console
helm install rel-jupyter nvidia-charts/jupyter -f values.yaml
```
