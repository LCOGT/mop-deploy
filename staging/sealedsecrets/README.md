# sealedsecrets

## Description

This package will replace all `v1.Secrets` with encrypted `SealedSecrets`.


## Usage

Clone this package:

```shell
kpt pkg get https://github.com/LCOGT/kpt-pkg-catalog/sealedsecrets
```

Or, if adding/updating secrets, update the package first:

```shell
kpt pkg update
```

The certificate used to encrypt these secrets is periodically updated, so downstream
consumers of this package should always run `kpt pkg update` before doing anything
else to fetch that.

Create a `Secret` that needs to be encrypted:

```shell
cat <<EOF > secret-name.yaml
apiVersion: v1
kind: Secret
metadata:
  name: name
  namespace: namespace
stringData:
  SECRET_TOKEN: "blha blha blha"
EOF
```

File name does not matter. It can be in any YAML file under this directory.

Then run:

```shell
kpt fn render
```

The `Secret` will be replaced with `SealedSecret` (in the same file).

Now add this file to `kustomization.yaml`, if it's not already there:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ./fix-name-refs/
  ...
  - ./secret-name.yaml # <--- Here

```

This Kustomization can now be used in others to apply these secrets. Name references
to these the orignal Secret (i.e. pre-hashed names) will also be updated automatically.

These files are safe to check into Git (after `kpt fn render` has proccesed them and converted all `v1.Secret` resources into `SealedSecret` resources).
