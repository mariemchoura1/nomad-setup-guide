# nomad-setup-guide
Guide for installing Nomad, creating a local cluster, and deploying an example application.
## 1. Install Nomad
Install the required packages.
1. **Installthe required packages:**
   sudo apt-get update && \
   sudo apt-get install wget gpg coreutils
2. **Add the HashiCorp GPG key:**
   wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
3. **Add the official HashiCorp Linux repository:**
   echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
 4. **Update and install Nomad:**
    sudo apt-get update && sudo apt-get install nomad
5. **Verify the installation:**
   nomad -v
## 2. Create a cluster
**Prerequisites**
Docker Desktop installed locally
Nomad supports many [task drivers](https://developer.hashicorp.com/nomad/docs/drivers) but the example application in this tutorial uses Docker. Docker Desktop provides the Docker engine, the CLI client, and a graphical interface. Other third-party options are available for running Docker on your machine but we recommend Docker Desktop for the simplicity and ease of use.

Nomad uses a socket file to communicate with Docker and [by default](https://developer.hashicorp.com/nomad/docs/drivers/docker#endpoint), this file is /var/run/docker.sock on Linux, Mac, and other Unix platforms, and /./pipe/docker_engine on Windows. Docker Desktop creates a symlink to this socket during installation.

If the Nomad client fails to detect the Docker driver with the error Constraint "missing drivers": 1 nodes excluded by filter, the issue may be this missing symlink. Additional information about this symlink for Mac and [Windows](https://docs.docker.com/desktop/windows/permission-requirements/) is available.

