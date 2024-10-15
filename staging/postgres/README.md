# cnpg-postgres

## Description

This package provides a bare-bones [`postgresql.cnpg.io/Cluster`](https://cloudnative-pg.io/documentation/1.24/cloudnative-pg.v1/#postgresql-cnpg-io-v1-Cluster)
that you can build upon and use in other packages.

## Usage

Clone this package:

```shell
kpt pkg get https://github.com/LCOGT/kpt-pkg-catalog/cnpg-postgres postgres
```

Customize `postgres.yaml`:

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: db
spec:
  ...
```

And then render to update resources:

```shell
kpt fn render
```

The operator will automatically create a set of `v1.Services` with the following names:

- `<cluster-name>-rw`: Points to the primary instance of the cluster (read/write).

Following services can be enabled by removing them from `spec.managed.services.disabledDefaultServices`:

- `<cluster-name>-ro`: Points to the replicas, where available (read-only).
- `<cluster-name>-r`: Points to any PostgreSQL instance in the cluster (read).

These names should be used to connect to the database from other Pods.

This package is also a Kustomization, so it can also be referenced by other
Kustomizations:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ./postgres/
```
