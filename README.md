# [Agones Platform](https://agones.dev) Operator

## Building the Operator

### Download the Operator SDK

```bash
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/operator-sdk/latest/operator-sdk-linux-x86_64.tar.gz
```

### Untar the bundle
```bash
tar -xzf operator-sdk-linux-x86_64.tar.gz
```

### Copy the `operator-sdk` binary into a location in your path
```bash
cp x86_64/operator-sdk /usr/local/bin/
```

###  Create a project directory
```bash
mkdir agones-operator
cd agones-operator

```

### Initialize the operator project with they helm plugin type
```bash
operator-sdk init --plugins helm --domain agones.dev
```

### Create the API
```bash
operator-sdk create api --helm-chart=agones --helm-chart-repo=https://agones.dev/chart/stable
```
It's okay to ignore the warnings.

### Now you should have a directory structure like this:
```tree
.
├── config
│   ├── crd
│   │   ├── bases
│   │   │   └── charts.agones.dev_agones.yaml
│   │   └── kustomization.yaml
│   ├── default
│   │   ├── kustomization.yaml
│   │   ├── manager_auth_proxy_patch.yaml
│   │   └── manager_config_patch.yaml
│   ├── manager
│   │   ├── kustomization.yaml
│   │   └── manager.yaml
│   ├── manifests
│   │   └── kustomization.yaml
│   ├── prometheus
│   │   ├── kustomization.yaml
│   │   └── monitor.yaml
│   ├── rbac
│   │   ├── agones_editor_role.yaml
│   │   ├── agones_viewer_role.yaml
│   │   ├── auth_proxy_client_clusterrole.yaml
│   │   ├── auth_proxy_role_binding.yaml
│   │   ├── auth_proxy_role.yaml
│   │   ├── auth_proxy_service.yaml
│   │   ├── kustomization.yaml
│   │   ├── leader_election_role_binding.yaml
│   │   ├── leader_election_role.yaml
│   │   ├── role_binding.yaml
│   │   ├── role.yaml
│   │   └── service_account.yaml
│   ├── samples
│   │   ├── charts_v1alpha1_agones.yaml
│   │   └── kustomization.yaml
│   └── scorecard
│       ├── bases
│       │   └── config.yaml
│       ├── kustomization.yaml
│       └── patches
│           ├── basic.config.yaml
│           └── olm.config.yaml
├── Dockerfile
├── helm-charts
│   └── agones
│       ├── certs
│       │   ├── allocator
│       │   │   ├── client-ca
│       │   │   │   └── ca.crt
│       │   │   ├── server.crt
│       │   │   └── server.key
│       │   ├── cert.sh
│       │   ├── server.crt
│       │   └── server.key
│       ├── Chart.yaml
│       ├── defaultfeaturegates.yaml
│       ├── README.md
│       ├── scripts
│       │   └── delete_agones_resources.sh
│       ├── templates
│       │   ├── controller.yaml
│       │   ├── crds
│       │   │   ├── fleetautoscaler.yaml
│       │   │   ├── fleet.yaml
│       │   │   ├── gameserverallocationpolicy.yaml
│       │   │   ├── gameserverset.yaml
│       │   │   ├── _gameserverspecschema.yaml
│       │   │   ├── _gameserverstatus.yaml
│       │   │   ├── gameserver.yaml
│       │   │   └── k8s
│       │   │       ├── _io.k8s.api.core.v1.PodTemplateSpec.yaml
│       │   │       └── _io.k8s.apimachinery.pkg.apis.meta.v1.ObjectMeta.yaml
│       │   ├── extensions-deployment.yaml
│       │   ├── extensions.yaml
│       │   ├── _helpers.tpl
│       │   ├── hooks
│       │   │   ├── pre_delete_hook.yaml
│       │   │   ├── sa.yaml
│       │   │   └── scripts.yaml
│       │   ├── NOTES.txt
│       │   ├── pdb.yaml
│       │   ├── ping.yaml
│       │   ├── priority-class.yaml
│       │   ├── service
│       │   │   └── allocation.yaml
│       │   ├── serviceaccounts
│       │   │   ├── controller.yaml
│       │   │   └── sdk.yaml
│       │   ├── service-monitor.yaml
│       │   ├── service.yaml
│       │   └── tests
│       │       └── test-runner.yaml
│       └── values.yaml
├── Makefile
├── PROJECT
└── watches.yaml
```

### `make` the `agones` operator image and push it to quay.io

Things to note:
1. If you're a podman user you may need to do a `dnf install podman-docker` before building.
2. You may need to login to registry.redhat.io to pull the `registry.redhat.io/openshift4/ose-helm-operator` image. If you do not already have a login you can sign up for one on [Red Hat Developer](https://developer.redhat.com) for free.

Example:
```bash
make docker-build docker-push IMG=quay.io/agones-operator/agones:v1.32.0
```

### Bundle the operator for use with [Operator Lifecycle Manager](https://docs.openshift.com/container-platform/4.12/operators/understanding/olm/olm-understanding-olm.html) (OLM)

```bash
make bundle IMG=quay.io/agones-operator/agones:v1.32.0
```
Fill in all the details. The output should be a `bundle` directory with the following contents

```tree
bundle
├── manifests
│   ├── agones-operator.clusterserviceversion.yaml
│   ├── agones-operator-controller-manager-metrics-service_v1_service.yaml
│   ├── agones-operator-metrics-reader_rbac.authorization.k8s.io_v1_clusterrole.yaml
│   └── charts.agones.dev_agones.yaml
├── metadata
│   └── annotations.yaml
└── tests
    └── scorecard
        └── config.yaml
```
### Make the bundle image and push it to your image repo.

Example:
```bash
make bundle-build BUNDLE_IMG=quay.io/agones-operator/agones-operator:v1.32.0
```
```bash
podman push quay.io/agones-operator/agones-operator:v1.32.0
```

## Deploy Operator on OpenShift

```bash
operator-sdk run bundle -n agones-system quay.io/agones-operator/agones-operator:v1.32.0
```

The output should look like this:
```bash
INFO[0004] Creating a File-Based Catalog of the bundle "quay.io/agones-operator/agones-operator:v1.32.0"
INFO[0005] Generated a valid File-Based Catalog
INFO[0007] Created registry pod: quay-io-agones-operator-agones-operator-v1-30-0
INFO[0007] Created CatalogSource: agones-operator-catalog
INFO[0007] OperatorGroup "operator-sdk-og" created
INFO[0007] Created Subscription: agones-operator-v0-0-1-sub
INFO[0009] Approved InstallPlan install-flkhj for the Subscription: agones-operator-v0-0-1-sub
INFO[0009] Waiting for ClusterServiceVersion "agones-system/agones-operator.v0.0.1" to reach 'Succeeded' phase
INFO[0009]   Waiting for ClusterServiceVersion "agones-system/agones-operator.v0.0.1" to appear
INFO[0021]   Found ClusterServiceVersion "agones-system/agones-operator.v0.0.1" phase: Pending
INFO[0023]   Found ClusterServiceVersion "agones-system/agones-operator.v0.0.1" phase: InstallReady
INFO[0024]   Found ClusterServiceVersion "agones-system/agones-operator.v0.0.1" phase: Installing
INFO[0034]   Found ClusterServiceVersion "agones-system/agones-operator.v0.0.1" phase: Succeeded
INFO[0034] OLM has successfully installed "agones-operator.v0.0.1"
```


