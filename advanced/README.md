# Advanced Kubernetes

In this event we'll go through a few more advanced uses of Kubernetes.

## Helm

Helm is described as "The package manager for Kubernetes". Essentially, Helm
allows you to distribute a bundle of Kubernetes object definitions which can be
deployed as a single unit, with a lot of potential for customisation (via
templating).

### Setup

Helm is a single CLI program, see [the installation page](https://helm.sh/docs/intro/install/)
for details.

### Deploy an application

Helm uses the Kubernetes API (in a manner similar to `kubectl`) to manage
deployments. Helm v3 (the current major release) requires no management tools
inside your cluster and uses Kubernetes objects only.

1. Add the Bitnami chart repository:

```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Helm charts are pulled from repositories - Bitnami provide a wide variety of
charts for deploying popular applications.

2. Install the WordPress chart:

```
helm install my-wp bitnami/wordpress \
    --set wordpressUsername=admin \
    --set wordpressPassword=hunter2 \
    --set mariadb.auth.rootPassword=hunter22 \
    --set service.type=ClusterIP \
    --set ingress.enabled=true \
    --set ingress.hostname=wordpress
```

3. Check the deployment:

```
helm list
```

4. Scale the number of WordPress pods:

```
helm upgrade my-wp bitnami/wordpress --reuse-values --set replicaCount=3
```

See [the chart README](https://github.com/bitnami/charts/tree/master/bitnami/wordpress)
for an explanation of all possible parameters.

5. Uninstall the release:

```
helm uninstall my-wp
```

## Create a Helm chart

1. Create the Helm chart structure:

```
helm create whoami
```

2. Modify the `Chart.yaml`, `values.yaml` (default values) and templates to
deploy `whoami`. It doesn't need a service account either. The ingress template
provides a level of customisation not needed for the whoami app. Be sure to
check the `NOTES.txt` template, as it references some of the values!

3. Create a values file:

```
replicaCount: 3
ingress:
  enabled: true
  hosts: [my-whoami]
```

4. Create a release:

```
helm install whoami . -f values.test.yaml
```

5. Uninstall the release:

```
helm uninstall whoami
```

## GitOps with Flux 2

### Setup

Like Helm, Flux uses a CLI for management. This CLI manages the installation of
the Flux components. See [the installation page](https://toolkit.fluxcd.io/guides/installation/)
for details.

### Install Flux into your cluster

We'll be following [the image automation guide](https://toolkit.fluxcd.io/guides/image-update/)
to set up Flux.

1. Export GitHub credentials ([create a personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) first):

```
export GITHUB_USER=<username>
export GITHUB_TOKEN=<token>
```

2. Bootstrap Flux:

```
flux bootstrap github \
    --components-extra=image-reflector-controller,image-automation-controller \
    --owner=$GITHUB_USER \
    --repository=my-gitops \
    --branch=master \
    --path=k8s \
    --personal
```

This will create a Git repo, commit the Flux components (under the path `k8s/`
in the repo) and install Flux into your cluster.

3. Delete and re-create the generate deploy key from the repo (making it have
   read-write capabilities). To get the public key, do
   `kubectl -n flux-system get secret flux-system -o go-template='{{index .data "identity.pub"}}' | base64 -d`

### Deploy whoami with Helm Controller CRD's

1. Create a `HelmRepository` object:

_TBC_
