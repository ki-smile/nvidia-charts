# matlab

Mathworks Matlab chart to be used on DGX, as standard create a disk with "100Gi" mounted in /data and using one nvidia GPU.

## TL;DR

```console
helm repo add nvidia-charts https://ki-smile.github.io/nvidia-charts
helm repo update
helm install matlab nvidia-charts/matlab
```

## Add Helm repo

Before we can install charts from a repo we need to add it and fetch the information:

```console
helm repo add nvidia-charts https://ki-smile.github.io/nvidia-charts
helm repo update
```

## Installing the Chart

To install the chart with the release name `rel-matlab`. The idea with the Helm release concept is that you can have multiple installations of the charts in the same namespace.

```console
helm install rel-matlab nvidia-charts/matlab
```

## Scale down

When you don't need the resources (the nvidia-gpu's is a limited resource in the cluster), scale down the deployment so it's possible for other to use it.

```console
kubectl scale deployment rel-matlab --replicas=0
```

## Scale up

When you want to continue the work on a scaled down release you can scale up the deployment again

```console
kubectl scale deployment rel-matlab --replicas=1
```

## Uninstalling the Chart

To uninstall the `rel-matlab` release/deployment and remove all resources (including the data)

```console
helm uninstall rel-matlab
```

The command removes all the Kubernetes components associated with the chart **including persistent volumes** and deletes the release.

## Configuration

Read through the [values.yaml](./values.yaml)

If you want to override those values, you can put them in a yaml files, see [2gpu.yaml](./2gpu.yaml) for a sample using two gpu's instead of one.

```console
helm install rel-matlab nvidia-charts/matlab -f values.yaml
```
