# set-lco-karpenter-attrs

## Description

This package provides a [Kustomize `Component`](https://github.com/kubernetes/enhancements/tree/master/keps/sig-cli/1802-kustomize-components)
that can be used to set [`karpenter.lco.earth/provisioner-name` scheduling attributes](https://github.com/LCOGT/internal-docs/wiki/Scheduling-K8s-Workloads)
in other Kustomizations.

Only the following KRM kinds will be patched:

- `v1.Pod`
- `apps/v1.Deployment`
- `apps/v1.StatefulSet`
- `apps/v1.ReplicaSet`
- `apps/v1.DaemonSet`
- `batch/v1.Job`
- `batch/v1.CronJob`
- `dragonflydb.io/v1alpha1.Dragonfly` (Redis)
- `postgresql.cnpg.io/v1.Cluster`
- `rabbitmq.com/v1beta1.RabbitmqCluster`

All others kinds will not be patched by this Component and must be handled manually.

## Usage

Clone this package:

```shell
kpt pkg get https://github.com/LCOGT/kpt-pkg-catalog/set-lco-karpenter-attrs
```

Edit `package-context.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kptfile.kpt.dev
  annotations:
    config.kubernetes.io/local-config: "true"
data:
  provisioner: dev # <- Change this to "prod" if running production workloads
```

And then render to update resources:

```shell
kpt fn render
```

Now you can reference this `Component` from other Kustomizations to patch
any resources defined there. For example:


```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ./something-that-should-be-patch.yaml

components:
  - ./set-lco-karpenter-attrs/ # <- # Add this to your Kustomization
```

If you ever want to change the provisioner, just edit `package-context.yaml` and
re-run `kpt fn render`.

### Override w/ Annoation

If a resources is annoated with `set-karpenter-attrs.kpt.lco.earth/provisioner-name`:

```yaml
apiVersion: something
kind: Something
metadata:
  name: something
  annotations:
    set-karpenter-attrs.kpt.lco.earth/provisioner: "provisioner-name"
```

Then that will take precendence over the one defined in `package-context.yaml`.

This allows for more fine-grained control over which provisioner a particular resource
should use.

### Updates

Support for new KRMs kinds may be added from time to time. If you have already pulled this
package, you can pull in those changes with:

```shell
kpt pkg update
```
