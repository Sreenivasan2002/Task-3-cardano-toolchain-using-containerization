<a name="readme-top"></a>

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/your-username/cardano-toolchain">
  </a>

  <h1 align="center">Build Toolchain for Cardano using Containerization</h1>

  <p align="center">
    <a href="https://github.com/your-username/cardano-toolchain"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/your-username/cardano-toolchain">View Demo</a>
    ·
    <a href="https://github.com/your-username/cardano-toolchain/issues">Report Bug</a>
    ·
    <a href="https://github.com/your-username/cardano-toolchain/issues">Request Feature</a>
  </p>
</div>

<!-- ABOUT THE PROJECT -->
## About The Project
A containerized toolchain to build `cardano-node` and `cardano-cli`, including `libsodium`.

The project is designed to simplify the build process for Cardano by providing a container environment with all necessary dependencies configured.

<p align="right">(<a href="#readme-top">back to top</a>)</p>


<!-- GETTING STARTED -->
## Steps to Build the Containerfile
#### Setting the Base Image
```Dockerfile
FROM debian:stable-slim AS builder
