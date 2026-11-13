# cilium-extensions

Extra resources for Cilium that have to be installed after it, such as the CiliumNodeConfigs for ENI IP pre-allocation per node size.

**Homepage:** <https://github.com/giantswarm/cilium-extensions>

## Source Code

* <https://github.com/giantswarm/cilium-extensions>

## IP pre-allocation per node size

With `ipam.nodeConfigs.enabled` the chart renders one CiliumNodeConfig per entry of `ipam.nodeConfigs.sizes`, selected on the node's vCPU count. Install it after Cilium: the CiliumNodeConfig CRD only exists once the cilium-operator runs.

- The Cilium agent reads its CiliumNodeConfig when it starts. A new node gets it at once; a node that runs already keeps its settings until its agent restarts (a node roll, or a rollout restart of the cilium DaemonSet).
- Sizes not in the table keep Cilium's defaults. Values merge into the default table: a size is added or changed by setting it, and removed only by setting it to `null`.

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| image.name | string | `"giantswarm/cilium-extensions"` |  |
| image.tag | string | `""` |  |
| registry.domain | string | `"gsoci.azurecr.io"` |  |
| ipam.nodeConfigs.enabled | bool | `false` | Render the CiliumNodeConfigs. Only for Cilium in ENI mode with nodes Karpenter labels with their vCPU count (EKS). The CiliumNodeConfig CRD has to exist, so install this chart after Cilium. |
| ipam.nodeConfigs.nodeLabel | string | `"karpenter.k8s.aws/instance-cpu"` | Node label the sizes are selected on: the node's vCPU count. |
| ipam.nodeConfigs.sizes | object | `{"16":{"minAllocate":96,"preAllocate":16},"32":{"minAllocate":224,"preAllocate":16},"4":{"minAllocate":32,"preAllocate":16},"8":{"minAllocate":48,"preAllocate":16}}` | IPAM settings per vCPU count. minAllocate is the IPs a node of that size gets when it starts, preAllocate the free IPs the operator keeps on it after that. Keep minAllocate a multiple of 16 (a /28 prefix) and below the node pool's maxPods. 4, 8 and 16 vCPU are tested in roadmap#4433 and roadmap#4435; 224 on 32 vCPU adds two prefixes over the 192 tested, which were too tight for 188 pods. |

## CI values

`ci/ci-values.yaml` holds the values the CI render (`helm template` in app-build-suite) needs beyond the defaults, such as a `required` value without a default.
