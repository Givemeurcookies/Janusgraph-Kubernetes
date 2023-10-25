# Janusgraph Kubernetes manifests
A verbose reference repository for how to set up Kubernetes manifests for Janusgraph and it's dependencies. Can be used as a building block for test, development or production environments.

## How to set up
If you simply want a Janusgraph server up and running as fast as possible, you can copy the `operators` folder and the files under `manifests/*` and then pick a folder for the Janusgraph version you want to run (`0.6.4` or `1.0.0`). If you want, you can also run the different versions side by side.

## Required resources
- Scylladb is set up with 3 pods and 1GB of req/lim memory per member. This is fine for testing but adjust according to your needs.
- ElasticSearch is configured with 2 pods but not configured with any limits. As reference, an empty/idle ElasticSearch pod uses ~1.5GB of memory.
- Janusgraph is set to request 4GB of RAM and 8GB of limit.
## Operators and dependencies
ScyllaDb is used as the storage backend for Janusgraph and ElasticSearch as the mixed index.

The official operators for ElasticSearch and Scylladb is used, there has been made changes to the scylladb operator in order for the operator deployments to have node affinity towards nodes labeled with `kubernetes.io/arch=amd64` [https://kubernetes.io/docs/reference/labels-annotations-taints/#kubernetes-io-arch](kubernetes node label specification) but otherwise they are standard operators.
## Version differences
Janusgraph 1.0.0 and 0.6.4 are fairly similar, but they don't support the same indexing databases. Therefore I've put all the shared dependencies between the versions in `manifest/*` and versions required for specific versions of Janusgraph in `manifests/<janusgraph-semver>/*`.

## Multiarchitecture
The scylladb ARM does not support architecture, but you can still have certain nodes that run on AMD64 just for Scylladb. Node affinity has been added to the operators in order to support this.

## Healthchecks
Janusgraph does not have any suited liveness or readiness probes for Kubernetes [https://github.com/JanusGraph/janusgraph/issues/2807](look at issue #2807 in the Janusgraph repository for updates). It however crashes after around ~60 seconds if it can't get connection to either the storage or mixed index backend.

## Indexing and running groovy scripts on Janusgraph startup in Kubernetes
In the folder `config` you find two files. One which configures a custom config `janusgraph-config-configmap.yaml` that is inserted into the docker image and a second file which lets you run a Groovy script `janusgraph-groovy-indicies-configmap.yaml` upon start. The custom config is used to set up connection for the groovy script. These are simply config references so you will have to modify them to suit your own needs and also add authentication to for example ElasticSearch - also remember to comment out the volumes and configmaps in the Janusgraph deployment.