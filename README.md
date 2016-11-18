# BOSH Release for minio-dist
**Early Stage**

This bosh release will help to install a distributed version of minio, using bosh.

There is already a minio-bosh release but for single tenant only.

## Creating multiple pool

Minio S3 do not support scale-out instead you can create multiple pool up to 16 nodes.

As bosh can mount only 1 persistent disk by VM it didn't make sens to put multiple minio server process on the same VM.

So instead we create multiple pool, you can check an example in 
`templates/bosh-lite-v2-2clusters.yml` this uses bosh'ish 2.0



## Important !!!

Minio S3 do not support **scale-out**  or **shrink** and could only be up to 16 nodes.
This is not due to this bosh release but the desing from minio S3.


 > Minio is designed to never do that.
 
 > Dynamic addition and removal of nodes are essential when all the storage nodes are managed by minio. 
 > Such a design is too complex and restrictive when it comes to cloud native application. 
 
 > Old design is to give all the resources to the storage system and let it manage them efficiently between the tenants. 
 > Minio is different by design. 
 > It is designed to solve all the needs of a single tenant. Spinning minio per tenant is the job of external orchestration  
 > layer. Any addition and removal means one has to reblanace the nodes.  When Minio does it internally, it behaves like blackbox.It also adds significant complexity to Minio. 
 
> Minio is designed to be deployed once and forgotten. We dont even want users to be replacing failed drives and nodes. Erasure code has enough redundancy built it. By the time half the nodes or drives are gone, it is time to refresh all the hardware. If the user still requires rebalancing, one can always start a new minio server on the same system on a different port and simply migrate the data over. It is essentially what minio would do internally. Doing it externally means more control and visibility.

- https://github.com/abperiasamy


## Usage

To use this bosh release, first upload it to your bosh:

```
bosh target BOSH_HOST
git clone https://github.com/shinji62/minio-dist-boshrelease.git
cd minio-dist-boshrelease
bosh upload release releases/minio-dist/minio-dist-1.yml
```

For [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can use a manifest example `templates/bosh-lite-v2.yml`.
```
bosh deployment templates/bosh-lite-v2.yml
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
