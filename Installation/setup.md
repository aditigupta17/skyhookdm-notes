
# SkyhookDM Ceph Setup and Build

## Method 1: Using Docker + Popper

### Dev setup
Install [Docker](https://docs.docker.com/get-docker/) and [Popper](https://github.com/getpopper/popper/blob/master/docs/sections/getting_started.md#installation). Then:
```bash
popper run dev-init
```
The above clones ceph and creates symlinks to this project within the ceph tree so that the cls folder is copied to the right place.

> **NOTE**: Take a look at the [`.popper.yml`](https://github.com/uccross/skyhookdm-ceph-cls/blob/master/.popper.yml) file, which contains the definition of what the `dev-init` command does.

 **NOTE:** In case you encounter, ```ERROR: Unable to connect to the docker daemon.``` or some other issues, follow the steps below:


Create  an alias for `popper`
 ```bash
alias popper='docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $PWD:$PWD -w $PWD getpopper/popper:2.6.0'
```

> Test if `popper` works by running
>  ```bash
>  cd /tmp/
>  popper run -f wf.yml
>  ```
>  It should run the workflow successfully.

If the step above works successfully, then:
```bash
popper run dev-init
```
### Build the library
```bash
popper run build
```
> **NOTE**: Take a look at the [`.popper.yml`](https://github.com/uccross/skyhookdm-ceph-cls/blob/master/.popper.yml) file, which contains the definition of what the `build` command does.

###  Generate a rook-compatible docker image
```
popper run build-rook-img
```
> **NOTE**: Take a look at the [`.popper.yml`](https://github.com/uccross/skyhookdm-ceph-cls/blob/master/.popper.yml) file, which contains the definition of what the `build-rook-img` command does.

### Run custom tests
```
popper run test
```
> **NOTE**: Take a look at the [`.popper.yml`](https://github.com/uccross/skyhookdm-ceph-cls/blob/master/.popper.yml) file, which contains the definition of what the `test` command does.

### Run dev test queries
< Add commands >

### Interactive shell

Any of the commands used above can be executed in interactive mode. For example, to open a shell on the `build` step:
```
popper sh build
```
The above opens an interactive shell inside an instance of the [builder image](https://github.com/uccross/skyhookdm-ceph-cls/blob/master/docker/Dockerfile.builder), which is a pre-built image with all the dependencies needed to build the CLS.

## Method 2: Manual configuration
### Clone the repository:

```bash
CEPH_VERSION=luminous
git clone \  
  --recursive \  
  --shallow-submodules \  
  --depth 1 \  
  --branch skyhookdm-$CEPH_VERSION \  
  https://github.com/uccross/skyhookdm-ceph
```


### Build the repository:
```bash
docker run --rm -ti \  
  -e CMAKE_FLAGS='-DBOOST_J=16 -DWITH_PYTHON3=OFF' \  
  -e BUILD_THREADS=16 \  
  -v $PWD:/ws \  
  -w /ws \  
  uccross/skyhookdm-builder:luminous \  
    cls_tabular
```
Here, the `-DBOOST_J=16` setting is controls the number of build jobs used to build boost. The `BUILD_THREADS=16` controls the number of jobs for building SkyhookDM library.


The container should 	start and log the user in automatically as `root@container_id`. If it does not, 

```bash
# find the latest container and container_id
docker ps -a

# attach the latest container
docker attach <container_id>

# switch to working directory inside the container
cd ws/
```
