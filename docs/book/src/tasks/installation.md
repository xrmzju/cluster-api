# Installation

## Prerequisites

- Install and setup [kubectl] in your local environment.
- Install and/or configure a [management cluster]

## Setup Management Cluster

Cluster API requires an existing kubernetes cluster accessible via kubectl, choose one of the options below:

1. **Kind**

{{#tabs name:"kind-cluster" tabs:"AWS,Docker,vSphere"}}
{{#tab AWS}}

<aside class="note warning">

<h1>Warning</h1>

[kind] is not designed for production use, and is intended for development environments only.

</aside>

  ```bash
  kind create cluster --name=clusterapi
  export KUBECONFIG="$(kind get kubeconfig-path --name="clusterapi")"
  ```
{{#/tab }}
{{#tab Docker}}

<aside class="note warning">

<h1>Warning</h1>

[kind] is not designed for production use, and is intended for development environments only.

</aside>

<aside class="note warning">

<h1>Warning</h1>

The Docker provider is not designed for production use and is intended for development environments only.

</aside>

<aside class="note warning">

<h1>Docker Provider on MacOS</h1>

Instructions for using the Docker provider on MacOS will be added soon.

</aside>

  Because the Docker provider needs to access Docker on the host, a custom kind cluster configuration is required:

  ```bash
  cat > kind-cluster-with-extramounts.yaml <<EOF
kind: Cluster
apiVersion: kind.sigs.k8s.io/v1alpha3
nodes:
  - role: control-plane
    extraMounts:
      - hostPath: /var/run/docker.sock
        containerPath: /var/run/docker.sock
EOF
  kind create cluster --config ./kind-cluster-with-extramounts.yaml --name clusterapi
  export KUBECONFIG="$(kind get kubeconfig-path --name="clusterapi")"
  ```
{{#/tab }}
{{#tab vSphere}}

<aside class="note warning">

<h1>Warning</h1>

[kind] is not designed for production use, and is intended for development environments only.

</aside>

  ```bash
  kind create cluster --name=clusterapi
  export KUBECONFIG="$(kind get kubeconfig-path --name="clusterapi")"
  ```
{{#/tab }}
{{#/tabs }}


2. **Existing Management Cluster**

For production use-cases a "real" kubernetes cluster should be used with apropriate backup and DR policies and procedures in place.

```bash
export KUBECONFIG=<...>
```

3. **Pivoting**

Pivoting is the process of taking an initial kind cluster to create a new workload cluster, and then converting the workload cluster into a management cluster by migrating the Cluster API CRD's.


## Installation

Using [kubectl], create the components on the [management cluster]:

#### Install Cluster API

```bash
kubectl create -f {{#releaselink gomodule:"sigs.k8s.io/cluster-api" asset:"cluster-api-components.yaml" version:"0.2.x"}}
```

#### Install the Bootstrap Provider

{{#tabs name:"tab-installation-bootstrap" tabs:"Kubeadm"}}
{{#tab Kubeadm}}

Check the [Kubeadm provider releases](https://github.com/kubernetes-sigs/cluster-api-bootstrap-provider-kubeadm/releases) for an up-to-date components file.

```bash
kubectl create -f {{#releaselink gomodule:"sigs.k8s.io/cluster-api-bootstrap-provider-kubeadm" asset:"bootstrap-components.yaml" version:"0.1.x"}}
```

{{#/tab }}
{{#/tabs }}


#### Install Infrastructure Provider

{{#tabs name:"tab-installation-infrastructure" tabs:"AWS,Docker,vSphere"}}
{{#tab AWS}}

<aside class="note warning">

<h1>Action Required</h1>

For more information about credentials management, IAM, or requirements for AWS, visit the [AWS Provider Prerequisites](https://github.com/kubernetes-sigs/cluster-api-provider-aws/blob/master/docs/prerequisites.md) document.

</aside>

#### Install clusterawsadm

Download the latest binary of `clusterawsadm` from the [AWS provider releases] and make sure to place it in your path.

##### Create the components

Check the [AWS provider releases] for an up-to-date components file.

```bash
# Create the base64 encoded credentials using clusterawsadm.
# This command uses your environment variables and encodes
# them in a value to be stored in a Kubernetes Secret.
export AWS_B64ENCODED_CREDENTIALS=$(clusterawsadm alpha bootstrap encode-aws-credentials)

# Create the components.
curl -L {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-aws" asset:"infrastructure-components.yaml" version:"0.4.x"}} \
  | envsubst \
  | kubectl create -f -
```

{{#/tab }}
{{#tab Docker}}

Check the [Docker provider releases](https://github.com/kubernetes-sigs/cluster-api-provider-docker/releases) for an up-to-date components file.

```bash
kubectl create -f {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-docker" asset:"provider_components.yaml" version:"0.2.x"}}
```

{{#/tab }}
{{#tab vSphere}}

Check the [vSphere provider releases](https://github.com/kubernetes-sigs/cluster-api-provider-vsphere/releases) for an up-to-date components file.

For more information about prerequisites, credentials management, or permissions for vSphere, visit the [getting started guide](https://github.com/kubernetes-sigs/cluster-api-provider-vsphere/blob/master/docs/getting_started.md).

```bash
kubectl create -f {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-vsphere" asset:"infrastructure-components.yaml" version:"0.5.x"}}
```

{{#/tab }}
{{#/tabs }}


<!-- links -->
[kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[components]: ../reference/glossary.md#provider-components
[kind]: https://sigs.k8s.io/kind
[management cluster]: ../reference/glossary.md#management-cluster
[target cluster]: ../reference/glossary.md#target-cluster
[AWS provider releases]: https://github.com/kubernetes-sigs/cluster-api-provider-aws/releases
