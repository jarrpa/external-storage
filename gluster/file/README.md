# GlusterFS Volume Provisioner for Kubernetes 1.5+

[![Docker Repository on Quay](https://quay.io/repository/external_storage/glusterfile-provisioner/status "Docker Repository on Quay")](https://quay.io/repository/external_storage/glusterfile-provisioner)
```
quay.io/external_storage/glusterfile-provisioner:latest
```
## What is the GlusterFS Provisioner?

The GlusterFS Provisioner is an external provisioner which dynamically provision GlusterFS volumes  on demand. The persistent Volume Claim which has been requested with this external provisioner's identity ( for eg# `gluster.org/glusterfs`)  will be served by this provisioner.

[GlusterFS](https://github.com/gluster/glusterfs)
[heketi](https://github.com/heketi/heketi)
[gluster-kubernetes](https://github.com/gluster/gluster-kubernetes)

## Build GlusterFS Provisioner and container image

If you want to build the container from source instead of pulling the docker image, please follow below steps:

 Step 1: Build the provisioner binary
```
[root@localhost]# go build glusterfs-provisioner.go
```

Step 2:  Get GlusterFS Provisioner Container image
```
[root@localhost]# docker pull quay.io/external_storage/glusterfile-provisioner:latest
```

## Start Kubernetes Cluster

## Start GlusterFS Provisioner

The following example uses `gluster.org/glusterfs` as the identity for the instance and assumes kubeconfig is at `/root/.kube`. The identity should remain the same if the provisioner restarts. If there are multiple provisioners, each should have a different identity.

```
[root@localhost] docker run -ti -v /root/.kube:/kube -v /var/run/kubernetes:/var/run/kubernetes --privileged --net=host  glusterfs-provisioner  -master=http://127.0.0.1:8080 -kubeconfig=/kube/config -id=gluster.org/glusterfs
```

## Create a GlusterFS StorageClass

```
[root@localhost] kubectl create -f examples/class.yaml
```

The available storage class parameter are listed below:

```yaml
parameters:
    resturl: "http://127.0.0.1:8081"
    restuser: "admin"
    restsecretnamespace: "default"
    restsecretname: "heketi-secret"
```

* `resturl` : Gluster REST service/Heketi service url which provision GlusterFS volumes on demand. The general format should be `IPaddress:Port` and this is a mandatory parameter for GlusterFS dynamic provisioner. If Heketi service is exposed as a routable service in openshift/kubernetes setup, this can have a format similar to
`http://heketi-storage-project.cloudapps.mystorage.com` where the fqdn is a resolvable heketi service url.

* `restuser` : Gluster REST service/Heketi user who has access to create volumes in the Gluster Trusted Pool.

* `restsecretnamespace` + `restsecretname` : Identification of Secret instance that contains user password to use when talking to Gluster REST service. These parameters are optional, empty password will be used when both `restsecretnamespace` and `restsecretname` are omitted. The provided secret must have type "gluster.org/glusterfs".

## How to test:

### Create a claim

```
[root@localhost]# kubectl create -f examples/claim1.yaml
persistentvolumeclaim "claim1" created

[root@localhost]# kubectl get pvc
NAME      STATUS    VOLUME                                     CAPACITY   ACCESSMODES   STORAGECLASS   AGE
claim1    Bound     pvc-b7045edf-3a26-11e7-af53-c85b7636c232   1Gi        RWX           glusterfs     56s
[root@localhost]# kubectl get pv
NAME                                       CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM            STORAGECLASS   REASON    AGE
pvc-b7045edf-3a26-11e7-af53-c85b7636c232   1Gi        RWX           Delete          Bound     default/claim1   glusterfs               46s

[root@localhost]# kubectl get pvc,pv
NAME         STATUS    VOLUME                                     CAPACITY   ACCESSMODES   STORAGECLASS   AGE
pvc/claim1   Bound     pvc-b7045edf-3a26-11e7-af53-c85b7636c232   1Gi        RWX           glusterfs     1m

NAME                                          CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM            STORAGECLASS   REASON    AGE
pv/pvc-b7045edf-3a26-11e7-af53-c85b7636c232   1Gi        RWX           Delete          Bound     default/claim1   glusterfs               1m
```
