Building Openshift Contiainer Storage Operator for OKD
---

# Introduction

This guide shows you how to build OpenShift Container Storage and deploy on OLM.

Pre-requisites:

- Latest stable version of Golang is installed
- [Operator SDK is installed](https://github.com/operator-framework/operator-sdk/releases)
- [OPM is downloaded and in your path](https://github.com/operator-framework/operator-registry/releases)

# Building

There are a few images that we will need to build.

- OpenShift Container Storage (OCS) Operator
- Operator Bundle
- Operator Index

### 1. Building the Operator

Clone the operator Github repo

```bash
git clone https://github.com/openshift/ocs-operator.git && cd ./ocs-operator
```

Checkout the desired branch

```bash
git checkout release-4.6
```

Set your desired environment variables (more can be found in the /hack scripts)

```bash
export REGISTRY=quay.io
export IMAGE_TAG=4.6.0
export REGISTRY_NAMESPACE=my-org-or-username
```

Build the OCS-Operator

```bash
make ocs-operator
```

Push the OCS operator to your registry

```bash
docker push $REGISTRY/$REGISTRY_NAMESPACE/ocs-operator:$IMAGE_TAG
```

Generate the latest ClusterServiceVersion files

```bash
make gen-latest-csv
```

Make and push the bundle image

```bash
make operator-bundle
```

```bash
docker push $REGISTRY/$REGISTRY_NAMESPACE/ocs-operator-bundle:$IMAGE_TAG
```

Next we need to create & push the index image

```bash
make operator-index
```

```bash
docker push $REGISTRY/$REGISTRY_NAMESPACE/ocs-operator-index:$IMAGE_TAG
```

### 3. Adding OCS to the Catalog

We next need to define a new CatalogSource & OperatorGroup for OCS. We can do this by running a simple multi-line command.

Create a namespace

```bash
oc create ns openshift-storage
```

Create the OperatorGroup

```bash
cat <<EOF | oc create -f -
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: openshift-storage-operatorgroup
  namespace: openshift-storage
spec:
  targetNamespaces:
    - openshift-storage
EOF
```

Next add the CatalogSource

```bash
cat <<EOF | oc create -f -
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: ocs-catalogsource
  namespace: openshift-marketplace
spec:
  sourceType: grpc
  image: ${REGISTRY}/${REGISTRY_NAMESPACE}/ocs-operator-index:${IMAGE_TAG}
  displayName: OpenShift Container Storage
  publisher: Red Hat
EOF
```

You now should be able to find OCS inside of the Catalog!