Building Local Storage Operator for OKD
---

# Introduction

This guide shows you how to slap the Local Storage Operator together to deploy on OLM.

Pre-requisites:

- Latest stable version of GoLang is installed
- [Operator SDK is installed](https://github.com/operator-framework/operator-sdk/releases)
- [OPM is downloaded and in your path](https://github.com/operator-framework/operator-registry/releases/tag/v1.16.1)

Some revisions of Local Storage Operator don't have all the build tools in the branch. You may need to do some branch hopping.

# Building

There are 3 major projects that we will need to build/push before we can deploy. We don't actually have to build these, but we can should we ever need to.

- [Local Storage Operator](https://github.com/openshift/local-storage-operator)
- Local Storage DiskMaker (included in operator repo)
- [Local Storage Static Provisioner](https://github.com/openshift/sig-storage-local-static-provisioner/tree/release-4.6) ( A fork from the official Kubernetes version )

### 1. Building the Local Storage Operator

Clone the Local Storage Operator into your local gopath.

```bash
go get github.com/openshift/local-storage-operator
```

Navigate into the cloned repo

```bash
cd $(go env GOPATH)/src/github.com/openshift/local-storage-operator
```

Checkout the desired branch

```bash
git checkout release-4.6
```

Set your desired environment variables

```bash
export REGISTRY=quay.io/my-org-or-username/
export VERSION=4.6.0
```

Next we are going to make and push our images. 

You should probably log into the container registry you plan to push to to avoid 403s

```bash
make push
```

Now verify that your images have successfully pushed and your cluster can access them over the network.

### 2. Building the Local Storage Static Provisioner

### TODO: Building the Local Static Provisioner [https://github.com/openshift/sig-storage-local-static-provisioner/](https://github.com/openshift/sig-storage-local-static-provisioner/tree/release-4.6)

### 3. Building the bundle

Next, we need to create a bundle image. The Bundle image is the base of the Operator. It contains custom resource definitions (CRDs) and other information needed by OKD to facilitate it's deployment. 

If you're missing the OPM-Bundle folder, download it from master.

Change into the bundle directory

```bash
cd opm-bundle
```

Modify the default file to fit your deployment and versions.

```bash
vim manifests/local-storage-operator.clusterserviceversion.yaml
```

Modify the YML file to point to your newly built & pushed images. 

Here is an example:

[LSO CSV Example](_example/lso-csv-example.yml)

Build and tag your Bundle Image

```bash
docker build -f ./bundle.Dockerfile -t ${REGISTRY}local-storage-bundle:${VERSION} .
```

Push the image

```bash
docker push ${REGISTRY}local-storage-bundle:${VERSION}
```

### 4. Building the index

Next, we need to build our Index Image. This image will simply point to our bundle, but it could point to multiple. This Index is what OLM searches to find available Operators. 

Use the OPM CLI to build the index image. 

Your Index image also needs a version

```bash
export INDEX_VERSION=4.6.1
opm index add --bundles ${REGISTRY}local-storage-bundle:${VERSION} --tag ${REGISTRY}local-storage-index:${INDEX_VERSION} --container-tool docker
```

Next push the image

```bash
docker push ${REGISTRY}local-storage-index:${INDEX_VERSION}
```

### 5. Add CatalogSource to OLM

Next, we need to tell OLM to search our index image for our bundle. We do this by using a CatalogSource.

Modify the following file and save it as catalog-create-subscribe.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-local-storage
---
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: local-operator-group
  namespace: openshift-local-storage
spec:
  targetNamespaces:
  - openshift-local-storage
---
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: localstorage-operator-manifests
  namespace: openshift-local-storage
spec:
  sourceType: grpc
  # replace this with your index image
  image: quay.io/my-org-or-username/local-storage-index:4.6.1
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: local-storage-subscription
  namespace: openshift-local-storage
spec:
  channel: preview   # this is the default channel name defined in opm-bundle file
  name: local-storage-operator
  source: localstorage-operator-manifests
  sourceNamespace: openshift-local-storage
```

Apply the image using the OC CLI.

```yaml
oc apply -f catalog-create-subscribe.yaml
```

You should now be able to find the Local Storage Operator inside of the OpenShift Marketplace under the openshift-local-storage project!
