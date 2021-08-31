# Compiling a non-AVX Tensorflow v1.15 using Docker

**Context:** Starting with Tensorflow v1.6, TF [discontinued support](https://github.com/tensorflow/tensorflow/issues/18689) for older CPUs that don't have AVX-instruction support. However, it's not feasible to stick with the older v1.5 version, as this locks you to Python 3.6 (which hit EOL status in December 2021). To avoid this (in other words, to upgrade Tensorflow while still providing support for these older CPUs), it is necessary to build Tensorflow from source. (Additional context can be found [here](https://www.eggwall.com/2020/09/compiling-tensorflow-without-avx.html).)

[`spinalcordtoolbox`](https://github.com/spinalcordtoolbox/spinalcordtoolbox) uses Tensorflow for its `sct_deepseg_gm`, `sct_deepseg_sc`, and `sct_deepseg_lesion` scripts. Our goal is to [continue to provide access](https://github.com/spinalcordtoolbox/spinalcordtoolbox/issues/3509) to these tools for users with older CPUs. So, we've forked this repository and use the Docker images to build Tensorflow from source.

## Usage

1. Clone this repository.
2. Install `docker` and `docker-compose`.
3. Modify the file `ubuntu-18.04/docker-compose.yml` to set some high-level configuration values, such as Tensorflow version, Python version, Bazel version, and CPU architecture.
   - We've set some sensible [default values](https://docs.docker.com/compose/environment-variables/#substitute-environment-variables-in-compose-files) corresponding to the [tested configuration for Tensorflow 1.15.X](https://www.tensorflow.org/install/source#tested_build_configurations).
   - We've also set the GCC CPU architecture to `core2` to build Tensorflow without relying on the AVX instruction set. (Info on CPU microarchitectures can be found [here](https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html).)
4. Modify the file `build.sh` to configure the low-level build environment.
   - We've restricted the resource usage to accommodate personal computers, but you can change both `--local_ram_resources` and `--jobs` if you have a more capable workstation.
5. Build the docker image using `cd tensorflow/ubuntu-18.04 && sudo docker-compose build`
6. Run the docker image using `sudo docker-compose run tf`.
7. Wait a number of hours. Your wheel will be in the newly-created `wheels/` folder.

## Why fork?

The original [`docker-tensorflow-builder`](https://github.com/hadim/docker-tensorflow-builder) repository is in an unmaintained, read-only state. This fork contains a few fixes and tweaks to make the repository functional again. The original README.md description for the repository is preserved below.

> ># Compile Tensorflow on Docker
>
>Docker images to compile TensorFlow yourself.
>
>Tensorflow only provide a limited set of build and it can be challenging to compile yourself on certain configuration. With this `Dockerfile`, you should be able to compile TensorFlow on any Linux platform that run Docker.
>
>Compilation images are provided for Ubuntu 18.10, Ubuntu 16.04, CentOS 7.4 and CentOS 6.6.
>
>## Requirements
>
>- `docker`
>- `docker-compose`
>
>## Usage
>
>- Clone this repository:
>
>```bash
>git clone https://github.com/hadim/docker-tensorflow-builder.git
>```
>
>### TensoFlow CPU
>
>- Edit the `build.sh` file to modify TensorFlow compilation parameters. Then launch the build:
>
>```bash
>LINUX_DISTRO="ubuntu-16.04"
># or LINUX_DISTRO="ubuntu-18.04"
># or LINUX_DISTRO="centos-7.4"
># or LINUX_DISTRO="centos-6.6"
>cd "tensorflow/$LINUX_DISTRO"
>
># Set env variables
>export PYTHON_VERSION=3.6
>export TF_VERSION_GIT_TAG=v1.13.1
>export BAZEL_VERSION=0.19
>export USE_GPU=0
>
># Build the Docker image
>docker-compose build
>
># Start the compilation
>docker-compose run tf
>
># You can also do:
># docker-compose run tf bash
># bash build.sh
>```
>
>### TensorFlow GPU
>
>- Edit the `build.sh` file to modify TensorFlow compilation parameters. Then launch the build:
>
>```bash
>LINUX_DISTRO="ubuntu-16.04"
># or LINUX_DISTRO="ubuntu-18.04"
># or LINUX_DISTRO="centos-7.4"
># or LINUX_DISTRO="centos-6.6"
>cd "tensorflow/$LINUX_DISTRO"
>
># Set env variables
>export PYTHON_VERSION=3.6
>export TF_VERSION_GIT_TAG=v1.13.1
>export BAZEL_VERSION=0.19
>export USE_GPU=1
>export CUDA_VERSION=10.0
>export CUDNN_VERSION=7.5
>export NCCL_VERSION=2.4
>
># Build the Docker image
>docker-compose build
>
># Start the compilation
>docker-compose run tf
>
># You can also do:
># docker-compose run tf bash
># bash build.sh
>```
>
>---
>
>- Refer to [tested build configurations](https://www.tensorflow.org/install/source#tested_build_configurations) to know which `BAZEL_VERSION` you need.
>- Be patient, the compilation can be long.
>- Enjoy your Python wheels in the `wheels/` folder.
>- *Don't forget to remove the container to free the space after the build: `docker-compose rm --force`.*
>
>## Builds
>
>| Tensorflow | Python | Distribution | Bazel | CUDA | cuDNN | NCCL | Comment |
>| --- | --- | --- | --- | --- | --- | --- | --- |
>| v2.0.0-alpha0 | 3.6 | Ubuntu 18.10 | 0.20 | 10.0 | 7.5 | 2.4 | seg fault error  |
>| v2.0.0-alpha0 | 3.6 | Ubuntu 18.10 | 0.20 | - | - | - | OK |
>| v2.0.0-alpha0 | 3.6 | Ubuntu 16.04 | 0.20 | 10.0 | 7.5 | 2.4 | TODO |
>| v2.0.0-alpha0 | 3.6 | Ubuntu 16.04 | 0.20 | - | - | - | TODO |
>| 1.9.0 | 3.6 | Ubuntu 16.04 | - | - | 0.19 | - | OK |
>| 1.9.0 | 3.6 | Ubuntu 16.04 | 9.0 | 0.19 | 7.1 | - | OK |
>| 1.9.0 | 3.6 | Ubuntu 16.04 | 9.1 | 0.19 | 7.1 | - | OK |
>| 1.9.0 | 3.6 | Ubuntu 16.04 | 9.2 | 0.19 | 7.1 | - | OK |
>| 1.9.0 | 3.6 | CentOS 6.6 | - | - | 0.19 | - | OK |
>| 1.9.0 | 3.6 | CentOS 6.6 | 9.0 | 0.19 | 7.1 | - | OK |
>| 1.9.0 | 3.6 | CentOS 6.6 | 9.1 | 0.19 | 7.1 | - | OK |
>| 1.9.0 | 3.6 | CentOS 6.6 | 9.2 | 0.19 | 7.1 | - | OK |
>
>## Authors
>
>- Hadrien Mary <hadrien.mary@gmail.com>
>
>## License
>
>MIT License. See [LICENSE](LICENSE).
>
