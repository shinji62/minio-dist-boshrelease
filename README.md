# BOSH Release for minio-dist
**Early Stage**

This bosh release will help to install a distributed version of minio.
There is already a minio-bosh release but for single tenant only.


## Important !!!

Minio S3 do not support **scale-out**  or **shrink** and could only be up to 16 nodes.
This is not due to this bosh release but the desing from minio S3.

author quote:
```
 Minio is designed to never do that. Dynamic addition and removal of nodes are essential when all the storage nodes are managed by minio. Such a design is too complex and restrictive when it comes to cloud native application. Old design is to give all the resources to the storage system and let it manage them efficiently between the tenants. Minio is different by design. It is designed to solve all the needs of a single tenant. Spinning minio per tenant is the job of external orchestration layer. Any addition and removal means one has to reblanace the nodes. When Minio does it internally, it behaves like blackbox. It also adds significant complexity to Minio. Minio is designed to be deployed once and forgotten. We dont even want users to be replacing failed drives and nodes. Erasure code has enough redundancy built it. By the time half the nodes or drives are gone, it is time to refresh all the hardware. If the user still requires rebalancing, one can always start a new minio server on the same system on a different port and simply migrate the data over. It is essentially what minio would do internally. Doing it externally means more control and visibility.
```




## Usage

To use this bosh release, first upload it to your bosh:

```
bosh target BOSH_HOST
git clone https://github.com/cloudfoundry-community/minio-dist-boshrelease.git
cd minio-dist-boshrelease
bosh upload release releases/minio-dist/minio-dist-1.yml
```

For [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can quickly create a deployment manifest & deploy a cluster. Note that this requires that you have installed [spruce](https://github.com/geofffranks/spruce).

```
templates/make_manifest warden
bosh -n deploy
```

For AWS EC2, create a single VM:

```
templates/make_manifest aws-ec2
bosh -n deploy
```

### Override security groups

For AWS & Openstack, the default deployment assumes there is a `default` security group. If you wish to use a different security group(s) then you can pass in additional configuration when running `make_manifest` above.

Create a file `my-networking.yml`:

``` yaml
---
networks:
  - name: minio-dist1
    type: dynamic
    cloud_properties:
      security_groups:
        - minio-dist
```

Where `- minio-dist` means you wish to use an existing security group called `minio-dist`.

You now suffix this file path to the `make_manifest` command:

```
templates/make_manifest openstack-nova my-networking.yml
bosh -n deploy
```

### Development

As a developer of this release, create new releases and upload them:

```
bosh create release --force && bosh -n upload release
```

### Final releases

To share final releases:

```
bosh create release --final
```

By default the version number will be bumped to the next major number. You can specify alternate versions:


```
bosh create release --final --version 2.1
```

After the first release you need to contact [Dmitriy Kalinin](mailto://dkalinin@pivotal.io) to request your project is added to https://bosh.io/releases (as mentioned in README above).
