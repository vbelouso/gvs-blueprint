# Generative Virtual Screening on OpenShift

The NVIDIA NIM Agent Blueprints for generative virtual screening shows how generative AI and accelerated NIM microservices can be used to design optimized small molecules smarter and faster.

In the repository you will find YAML manifests that will start up the NIMs for AlphaFold 2, MolMIM, and DiffDock.
Once the NIMs are started, you can use the example notebook in the `src` directory.

## Prerequisites

* [OpenShift CLI](https://docs.openshift.com/container-platform/latest/installing/installing_platform_agnostic/installing-platform-agnostic.html#cli-installing-cli_installing-platform-agnostic)
* You must be logged into an OpenShift Container Platform cluster.
* NVIDIA GPU Cloud (NGC) API key

## Get NVIDIA GPU Cloud (NGC) API key

You can get the key in two ways:

* Navigate to the [API Catalog](https://build.nvidia.com/nvidia/generative-virtual-screening-for-drug-discovery), ensure you're logged in, and click "Download Blueprint" to generate an API Key.

* Navigate to the [NVIDIA NGC hub](https://org.ngc.nvidia.com/setup/api-key), ensure you're logged in, and click "Generate API Key".

It can be convenient to save your API Key to a file for later use. Here, we'll assume you've saved your API Key to a plain text file called `~/ngc-api-key`.

## Deploy blueprint

### Creating a project

The following example uses the project name `gvs`. Please update it according to your requirements.

```shell
NAMESPACE="gvs"
oc new-project "${NAMESPACE}"
```

### Creating a NGC secret

```shell
oc create secret generic ngc-api-key --from-file=NGC_CLI_API_KEY="${HOME}/ngc-api-key"
```

### Creating a pull secret to access NVIDIA Container Registry

```shell
oc create secret docker-registry nvcr-io-pull-secret \
  --docker-server=nvcr.io \
  --docker-username='$oauthtoken' \
  --docker-password="$(cat ${HOME}/ngc-api-key)"
```

### Update RoleBindings

```shell
find deploy/openshift/ -name "*.yaml" -exec sed -i "s/<NAMESPACE>/${NAMESPACE}/g" {} +
```

## Deploy the NIMs

```shell
oc create -f deploy/openshift/
```

You should see container images pulling events, which can take up to several hours, depending on the speed of internet connection.
Once the containers are pulled, they will start.

> [!IMPORTANT]
> On first launch, Alphafold downloads a database of approximately **510 GB**.

Until the database is completely downloaded, the service does not work correctly.
Monitoring data loading is possible via endpoint `localhost:8000/v1/health/ready` until the next output is available

```json
{
  "status": "ready"
}
```

### Check the Status of the NIMs

Check **AlphaFold2** status from the container:

```shell
curl localhost:8000/v1/health/ready
```

Example output:

```json
{
  "status": "ready"
}
```

Check **DiffDock** status from the container:

```shell
curl localhost:8000/v1/health/ready
```

Example output:

```text
true
```

Check **MolMIM** status from the container:

```shell
curl localhost:8000/v1/health/ready
```

Example output:

```json
{
  "status": "ready"
}
```

### Interacting with the NIMs

The three NIMs are available via services:

* alphafold: 8000
* diffdock: 8000
* molmim: 8000

For a complete example, navigate to the [src](../src) directory where you'll find a Jupyter notebook with a complete generative virtual screening pipeline.
