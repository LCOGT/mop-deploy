# mop-deploy

MOP GitOps repository.

ArgoCD should be setup to watch and sync directories/files in this repository.


## Usage

Always make sure you are in the `devenv` shell so that pre-commit hooks validate
any changes:

```shell
nix develop --impure
```

If you use SSH keys to authenticate with Github, add this to your `~/.gitconfig` to rewrite URLs on the fly:

```gitconfig
[url "git@github.com:LCOGT"]
  insteadOf = https://github.com/LCOGT
```

### staging

Use the `staging/` directory to define the staging environment.

This is a Kustomization (i.e. has a `kustomization.yaml`).

The rendered output of this Kustomization is written to `output/staging/manifest.yaml`.

ArgoCD should watch `output/staging/` (instead of the Kustomization).

A `kustomize-build-staging` pre-commit hook is enabled in `flake.nix` that will ensure that the output is kept up to date on any changes to the Kustomization:

```nix
        ...
        config.devenv.shells.default = {
          pre-commit.hooks.kustomize-build-staging.enable = lib.mkForce enable;
          ...
        }
```

`staging/base` should be a copy of the `base` Kustomization in the application
repository cloned using `kpt`:

```shell
kpt pkg get https://github.com/LCOGT/app-repo/k8s/base staging/base
```

This Kpt package will be updated automatically by the CI/CD pipline to keep it in sync with the application repository.

Add it to `staging/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

components:
 - ./cd-set-images/

resources:
  - ./base/
```

`staging/cd-set-images` is a kustomize `Component` that's used by the CI/CD
pipline to automatically set the container image version to the latest built revision
from the application repository. Leave it be.

If you do not need a staging environment, feel free to delete the directory and
make sure `flake.nix` disables the `kustomize-build-staging` pre-commit hook:

```nix
        ...
        config.devenv.shells.default = {
          pre-commit.hooks.kustomize-build-staging.enable = lib.mkForce false;
          ...
        }
```

### prod

Similar to `staging/`, the `prod/` directory defines the production environment.

Make sure `kustomize-build-prod` pre-commit hook is enabled:

```nix
        ...
        config.devenv.shells.default = {
          pre-commit.hooks.kustomize-build-prod.enable = lib.mkForce true;
          ...
        }
```

`prod/base` should be a Kpt package pointing to `staging/base` in the deploy repo:

```shell
kpt pkg get https://github.com/LCOGT/app-deploy-repo/staging/base prod/base
```

`prod/cd-set-images` should be a Kpt package pointing to `staging/cd-set-images` in the deploy repo:

```shell
kpt pkg get https://github.com/LCOGT/app-deploy-repo/staging/cd-set-images prod/cd-set-images
```

Both of these will be updated automatically by the CI/CD pipline.

Add them to `prod/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

components:
 - ./cd-set-images/

resources:
  - ./base/
```
