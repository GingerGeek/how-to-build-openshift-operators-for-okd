Openshift Container Storage on OKD
---

# Introduction

Openshift Container Storage (OCS) on OKD is software-defined storage for containers, with the storage services running natively in containers.

It's especially powerful on bare metal deployments or situations where you don't have a cloud integration for provision of storage.

Underneath, OCS is an opinionated deployment of Rook (which in-turn is an opionated deployment of Ceph in K8s) and NooBa (an object gateway).

There are many components to OCS which fit together to give the rather pleasant developer experience.

Under Openshift, OCS is available within the OperatorHub for near-oneclick install.

Under OKD, you must build and install OCS manually which is a somewhat involved process.

At some point in the future, a version of OCS should end up in the OKD OperatorHub - please follow along with the OKD Working Group to track progress on that.

When not using cloud-backed storage (e.g EBS) you will need to install the Local Storage Operator which allows you to use local block devices for the internal Ceph cluster.

# Contents

| Document | Overview |
|-|-|
| [Building LSO](building-lso.md) | How to download the code repository and build the Local Storage Operator in preparation for installation |
| [Building OCS](building-ocs.md) | How to download the code repository and build the OCS operators in preparation for installation |
| [Installing LSO+OCS and intial setup](setup-ocs.md) | Once the operators are in an image registry, how to install them within your cluster and setup the storage services |


# Useful Resources
- [Configure the OpenShift Image Registry to use Container Storage](https://www.openshift.com/blog/configure-the-openshift-image-registry-backed-by-openshift-container-storage)
- [Uninstalling OpenShift Container Storage](https://access.redhat.com/documentation/en-us/red_hat_openshift_container_storage/4.6/html/deploying_and_managing_openshift_container_storage_using_ibm_power_systems/assembly_uninstalling-openshift-container-storage_ibm-power)(4.6)
- [Local Storage Operator Github](https://github.com/openshift/local-storage-operator)
- [OpenShift Container Storage Operator Github](https://github.com/openshift/ocs-operator)
- [OKD](https://www.okd.io/)
- [Rafael's useful Cheat sheet](https://gist.github.com/rafaeltuelho/111850b0db31106a4d12a186e1fbc53e)
