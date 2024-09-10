<a name="readme-top"></a>

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/Sreenivasan2002">
  </a>

  <h1 align="center">Toolchain for Cardano using Containerization</h1>

  <p align="center">
    <strong>By Sreenivasan S, Reg. No: 21BEC0256</strong>
  </p>
</div>

<!-- ABOUT THE PROJECT -->
## About The Project
This project provides a comprehensive **Containerfile** designed to serve as a build toolchain for setting up and compiling both the `cardano-node` and `cardano-cli` tools, along with their dependency, `libsodium`. These components are essential for running and managing a Cardano node, interacting with the blockchain, and conducting transactions.

By containerizing the build environment, the project ensures that all necessary dependencies are properly managed and versioned, making it easier for developers to work with a consistent setup across different systems. This approach not only simplifies the development process but also significantly reduces the potential for build and compatibility issues by leveraging containerization technology.

In the following sections, you'll find detailed steps to build the container, ensuring a stable and efficient environment to run the Cardano node and CLI tools.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->
## Steps to Build the Containerfile

#### Setting the Base Image
The first step in the container build process is setting the base image. We use the stable version of Debian with the following line:
```bash
FROM debian:stable-slim AS builder


#### Set environment variables
```ENV DEBIAN_FRONTEND noninteractive```

This prevents any interactive prompts during package installations.

```
ENV LC_ALL C.UTF-8
ENV LANG C.UTF-8
```
These lines set the character encoding for the local and system language.


#### Install build dependencies
```
RUN apt-get update -y
```
Updating the package repository metadata.

```
RUN apt-get install -y \
  python3 \
  python3-pip \
  automake \
  build-essential \
  pkg-config \
  libffi-dev \
  libgmp-dev \
  libssl-dev \
  libtinfo-dev \
  systemd \
  libsystemd-dev \
  libsodium-dev \
  libsodium23 \
  zlib1g-dev \
  npm \
  yarn \
  make \
  g++ \
  tmux \
  git \
  jq \
  wget \
  libncursesw5 \
  gnupg \
  libtool \
  autoconf
```
Installing build dependencies required for GHC, Cabal, libsodium(dependencies for cardano-node and CLI) and cardano-node


#### Install cabal
```
RUN wget https://downloads.haskell.org/~cabal/cabal-install-3.2.0.0/cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz \
  && tar -xf cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz \
  && rm cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz cabal.sig \
  && mv cabal /usr/bin/ \
  && cabal update
```
Downloading the cabal version 3.2 tar file. Then after extracting the fo;der we remove the downloaded archive. Then we move cabal to /usr/bin/ so that we can run the cabal command then.


#### Install GHC
```
RUN wget https://downloads.haskell.org/ghc/8.10.2/ghc-8.10.2-x86_64-deb9-linux.tar.xz \
  && tar -xf ghc-8.10.2-x86_64-deb9-linux.tar.xz \
  && rm ghc-8.10.2-x86_64-deb9-linux.tar.xz \
  && cd ghc-8.10.2 \
  && ./configure \
  && make install \
  && cd ..
```
Downloading GHC (Glasgow Haskell Compiler) version 8.10.2, extracts it, removes the downloaded archive, configures, installs, and sets up GHC.

#### Install libsodium
```
RUN git clone https://github.com/input-output-hk/libsodium \
  && cd libsodium \
  && git fetch --all --recurse-submodules --tags \
  && git tag \
  && echo git checkout 66f017f1 \
  && ./autogen.sh \
  && ./configure \
  && make \
  && make install
```

Clones the libsodium repository, fetches tags, checks out a specific tag - 66f017f1. Then we run autogen.sh and configure, builds the library, and installs it.


```
ENV LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH" 
ENV PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"
```
Setting up the environment variable "LD_LIBRARY_PATH" to include the /usr/local/lib directory, allowing the system to find dynamically linked libraries at runtime. Also, the environment variable "PKG_CONFIG_PATH" to include the /usr/local/lib/pkgconfig directory, allowing the system to find pkg-config files for libraries at build time.

#### Install cardano-node
```
RUN echo "Building tags/1.26.0..." \
  && echo tags/1.26.0 > /CARDANO_BRANCH \
  && git clone https://github.com/input-output-hk/cardano-node.git \
  && cd cardano-node \
  && git fetch --all --recurse-submodules --tags \
  && git tag \
  && git checkout tags/1.26.0
```
Clones the cardano-node repository, fetches tags, checks out a specific tag - 1.26.0. Note: This is a stable version! (You can change the tag to another stable version if you want to build the another version of cardano-node.)


```
WORKDIR /cardano-node/
RUN cabal configure --with-compiler=ghc-8.10.2 \
  && echo "package cardano-crypto-praos" >>  cabal.project.local \
  && echo "  flags: -external-libsodium-vrf" >>  cabal.project.local \
  && cabal build cardano-node cardano-cli
RUN mkdir -p /root/.local/bin/ \
  && cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.2/cardano-node-1.26.0/x/cardano-node/build/cardano-node/cardano-node /root/.local/bin/ \
  && cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.2/cardano-cli-1.26.0/x/cardano-cli/build/cardano-cli/cardano-cli /root/.local/bin/
```
We change the work directory to /cardano-node/ and then we configure cabal to use the GHC version 8.10.2. Then we add the cardano-crypto-praos package to the cabal.project.local file. Then we build the cardano-node and cardano-cli. Then we create a directory /root/.local/bin/ and copy the cardano-node and cardano-cli binaries to the directory. 


#### Starting a new build stage
```
FROM debian:stable-slim
```
From here we start a new stage build. We use multi stage build to reduce the size of the final image.(Otherwise the final image will be around 9GB)


#### Copy the binaries/libraries we've just built in the builder stage
```
COPY --from=builder /root/.local/bin/cardano-* /usr/local/bin/
COPY --from=builder /usr/local/lib/libsodium* /usr/local/lib/
```
Copies the binaries and libraries built in the previous "builder" stage to the corresponding locations in the new stage.

#### Install tools
```
RUN apt-get update && \
  apt-get install -y \
  curl        \
  dnsutils    \
  jq          \
  net-tools   \ 
  procps      \ 
  python3     \
  python3-pip \ 
  telnet      \
  tmux        \
  vim         \
  wget        \
  bc          \
  curl
```
Updating and installing some necessary tools that may be needed when running the final container.

#### Remove uneccesary packages
```
RUN rm -rf /var/lib/apt/lists/*
```
Removing uneccesary libraries and files that we we installed in the builder stage that we dont need anymore. 

#### Set the working directory to /app
```
WORKDIR /app
```
Changing the work directory to /app

The containerfile is ready now!

## Usage
#### Building the image from Dockerfile

```
git clone https://github.com/Sreenivasan2002/Task-3-cardano-toolchain-using-containerization
cd cardano-toolchain
docker build -t cardano-toolchain .
docker run -it cardano-toolchain
```

#### Pulling the image from Docker Hub

```
docker run -it sreenivasan2002/cardano-toolchain
```

Then you can use the cardano-node and cardano-cli commands inside container.





### Continuous Integration
# Cardano Toolchain

This repository contains the Docker and Kubernetes setup for the Cardano toolchain.

## Continuous Integration

GitHub Actions has been set up to automate the process of building the Docker image and pushing it to Docker Hub on every push or pull request to the `main` branch. The workflow configuration can be found at:


## Docker Hub

The Docker image is available on Docker Hub:

[Cardano Toolchain Image](https://hub.docker.com/r/sreenivasan2002/cardano-toolchain/tags)

## Kubernetes Deployment Configuration Files

Kubernetes deployment and service configuration files are available in the `k8s-deployment-and-service-configs` branch. Please check them out!

## Challenges Faced

1. **Choosing the Correct Dependencies:**  
   The first challenge was selecting the appropriate dependencies. After reviewing the GHCup, Haskell, libsodium, and Cabal documentation, I was able to identify the correct ones.

2. **Version Compatibility:**  
   Finding compatible versions of Cabal, GHC, and libsodium required multiple attempts. After trying different combinations, I found a set that worked together.

3. **Finding a Stable Version of Cardano-Node:**  
   I struggled to find a stable version of `cardano-node`. After exploring the GitHub repository's tags section, I found and tested various versions until I settled on version 1.26.0, which worked reliably.

4. **Reducing Docker Image Size:**  
   Initially, the Docker image was around 9GB. By implementing a multi-stage build, I managed to reduce the image size to approximately 600MB, optimizing storage and performance.

<!-- CONTACT -->
## Contact

Sreenivasan S - [@sreenivasan](https://linkedin.com/in/sreenivasan-s-sreenivit/) - sreenivasanskpy@gmail.com

Project Link: (https://github.com/Sreenivasan2002/Task-3-cardano-toolchain-using-containerization)


