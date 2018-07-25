# cmssw-docker

Dockerfiles for CMSSW

There are different sets of Dockerfiles in this repository:

- [standalone](standalone) images [![](https://images.microbadger.com/badges/image/clelange/cmssw.svg)](https://microbadger.com/images/clelange/cmssw)
- [cvmfs](cvmfs)-based images
- [slc6-cms](slc6-cms) images [![](https://images.microbadger.com/badges/image/clelange/slc6-cms.svg)](https://microbadger.com/images/clelange/slc6-cms) [![](https://images.microbadger.com/badges/version/clelange/cmssw-slc6-only.svg)](https://microbadger.com/images/clelange/slc6-cms)

The non-standalone images need a network connection, and can be slow, since CMSSW is loaded via the network. The advantage is that they are much smaller (few hundreds of MB) while the standalone images contain the full CMSSW release (>= 15 GB).

All sets of images are currently based on SLC6.

## Building containers

Change to the directory containing the Dockerfiles and execute one of the following commands. Mind that the `.` at the end is important.

### Standalone version

 Examples are given for different `CMSSW_VERSION` and `SCRAM_ARCH`:

Production releases:

```shell
docker build -f Dockerfile_production \
            -t cmssw:9_2_1 \
            --build-arg CMSSW_VERSION=CMSSW_9_2_1 \
            --build-arg SCRAM_ARCH=slc6_amd64_gcc530 \
            .
```

Patch releases:

```shell
docker build -f Dockerfile_patch \
            -t cmssw:7_1_25_patch5 \
            --build-arg CMSSW_VERSION=CMSSW_7_1_25 \
            --build-arg SCRAM_ARCH=slc6_amd64_gcc481 \
            --build-arg PATCH=patch5 \
            .
```

### CVMFS version

Since these containers load CMSSW via the network, any CMSSW version can be set up.

```shell
docker build -t cmssw-cvmfs .
```

### SLC6-CMS version

This image does not know about CMSSW, it is only an SLC6 image with some additional packages installed. CVMFS needs to be mounted as volume (see below). A `Makefile` is available to build the image:

```shell
make
make docker_push
```

## Running containers

All images are available in the CERN GitLab container registry as well: [https://gitlab.cern.ch/clange/cmssw-docker/container_registry](https://gitlab.cern.ch/clange/cmssw-docker/container_registry)

In order to use these images, the run commands below need to be changed prepending `gitlab-registry.cern.ch/clange/cmssw-docker`.

### Standalone version

Currently supported for automatic CMSSW setup are `bash` and `zshrc`.

`bash`:

```shell
docker run --rm -it clelange/cmssw:9_2_1 /bin/bash
```

`zsh`:

```shell
docker run --rm -it cmssw:9_2_1 /bin/zsh
```

### CVMFS version

Setting up CVMFS uses `fuse`, which needs special rights from docker:

```shell
docker run --rm --cap-add SYS_ADMIN --device /dev/fuse -it cmssw-cvmfs /bin/bash
```

Alternatively, one can also just use `--privileged` instead of `--cap-add SYS_ADMIN --device /dev/fuse`, see the [Docker run reference](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities).

If you get an error similar to:

```shell
/bin/sh: error while loading shared libraries: libtinfo.so.5: failed to map segment from shared object: Permission denied
```

you need to turn off SElinux security policy enforcing:

```shell
sudo setenforce 0
```

This can be changed permanently by editing `/etc/selinux/config`, setting `SELINUX` to `permissive` or `disabled`.

### SLC6-only version

On a machine that has `/cvmfs` mounted (and available to the docker process):

```shell
docker run --rm -it -v /cvmfs:/cvmfs clelange/cmssw-slc6-only /bin/zsh
```

On CERN OpenStack (see [OpenStack CVMFS documentation](http://clouddocs.web.cern.ch/clouddocs/containers/tutorials/cvmfs.html)):

```shell
docker run --rm -it --volume-driver cvmfs -v cms.cern.ch:/cvmfs/cms.cern.ch clelange/cmssw-slc6-only /bin/zsh
```

and for the CMS OpenData extend this to:

```shell
docker run --rm -it --volume-driver cvmfs -v cms.cern.ch:/cvmfs/cms.cern.ch -v cms-opendata-conddb.cern.ch:/cvmfs/cms-opendata-conddb.cern.ch clelange/cmssw-slc6-only /bin/zsh
```

## Grid proxy

If you would like to be able to get a voms/grid proxy, mount your `.globus` directory by adding `-v ~/.globus:/home/cmsbld/.globus` to your `docker run` command.
