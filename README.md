# BOSH Release for Minio

**Early Stage**

This release will help to install a distributed version of [Minio](https://minio.io/).

This release use the new **[bosh cli](https://github.com/cloudfoundry/bosh-cli)**

```
$ bosh -d minio deploy manifests/example.yml --vars-store /tmp/minio-creds.yml
```

## Examples
There is 2 examples one with only one cluster `manifests/example.yml` and one with 2 clusters `manifests/example-2cluster.yml`





## Important !!!

Minio S3 do not support **scale-out** or **shrink** and could only be up to 16 nodes. This is not due to this release but the design of Minio.

 > Minio is designed to never do that.

 > Dynamic addition and removal of nodes are essential when all the storage nodes are managed by minio.
 > Such a design is too complex and restrictive when it comes to cloud native application.

 > Old design is to give all the resources to the storage system and let it manage them efficiently between the tenants.
 > Minio is different by design.
 > It is designed to solve all the needs of a single tenant. Spinning minio per tenant is the job of external orchestration
 > layer. Any addition and removal means one has to reblanace the nodes.  When Minio does it internally, it behaves like blackbox.It also adds significant complexity to Minio.

> Minio is designed to be deployed once and forgotten. We dont even want users to be replacing failed drives and nodes. Erasure code has enough redundancy built in. By the time half the nodes or drives are gone, it is time to refresh all the hardware. If the user still requires rebalancing, one can always start a new minio server on the same system on a different port and simply migrate the data over. It is essentially what minio would do internally. Doing it externally means more control and visibility.

- https://github.com/abperiasamy

You can create multiple instance groups of up to 16 instances to create several pools.

## Smoke tests

smoke-tests job is a very simple way to verify that your deployment is working. For each node it:

* Creates bucket
* Uploads file
* Reads file
* Deletes bucket

Just adds errand to your deployment to run smoke tests:

```yaml
- name: smoke-tests
  instances: 1
  lifecycle: errand
  azs: [z1]
  jobs:
  - name: smoke-tests
    release: minio-dist
  vm_type: default
  stemcell: default
  networks:
  - name: default
```
To run the errand
```
bosh run-errand smoke-tests -d minio
```

If you have multiple pools within the same deployment you can use links to configure smoke test job.

### Minio versions

| bosh release tag | Minio tag | Release
| ----------| -------- | -------- |
|v1|https://github.com/minio/minio/commits/4098025||
|v2|https://github.com/minio/minio/commits/5c9a95d||
|v3|https://github.com/minio/minio/commits/29d72b84|RELEASE.2016-12-13T17-19-42Z|
|v4|https://github.com/minio/minio/commits/29b49f93|RELEASE.2017-01-25T03-14-52Z|
