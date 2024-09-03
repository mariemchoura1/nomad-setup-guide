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

## 2. Clone the code repository
The example repository includes Terraform configurations for starting a cluster on the cloud as well as the jobspec files for the example application.

1. **Clone the code repository:**
git clone https://github.com/hashicorp-education/learn-nomad-getting-started.git

2. **Navigate to the repository folder:**
cd learn-nomad-getting-started

3. **Check out the v1.1 tag of the repository as a local branch named nomad-getting-started:**
git checkout -b nomad-getting-started v1.1

## 3. Create a cluster
**Prerequisites**
Docker Desktop installed locally
Nomad supports many [task drivers](https://developer.hashicorp.com/nomad/docs/drivers) but the example application in this tutorial uses Docker. Docker Desktop provides the Docker engine, the CLI client, and a graphical interface. Other third-party options are available for running Docker on your machine but we recommend Docker Desktop for the simplicity and ease of use.

Nomad uses a socket file to communicate with Docker and [by default](https://developer.hashicorp.com/nomad/docs/drivers/docker#endpoint), this file is "/var/run/docker.sock" on Linux, Mac, and other Unix platforms, and "/./pipe/docker_engine" on Windows. Docker Desktop creates a symlink to this socket during installation.

If the Nomad client fails to detect the Docker driver with the error Constraint "missing drivers": 1 nodes excluded by filter, the issue may be this missing symlink. Additional information about this symlink for Mac and [Windows](https://docs.docker.com/desktop/windows/permission-requirements/) is available.

Open your terminal and start the development agent. This creates a Nomad cluster of one node that acts as both the server and client.

The -bind flag set to 0.0.0.0 instructs Nomad to listen on all network interfaces present on the machine. The -network-interface flag instructs Nomad to use a network interface other than the default loopback interface (localhost). In this example, Nomad automatically gets the network interface attached to the default route on the machine with the GetDefaultInterfaces function but you can also provide the name of an interface. This flag is necessary for the example application as each service runs in a different container and cannot reach the other services on the loopback address.

1. **Leave Nomad running in this terminal session:**
sudo nomad agent -dev \
  -bind 0.0.0.0 \
  -network-interface='{{ GetDefaultInterfaces | attr "name" }}'
  
2. **In a second terminal session, export the cluster address:**
export NOMAD_ADDR=http://localhost:4646

## 4. Verify connectivity
nomad node status

## 5. Browse the web UI
Navigate to the Nomad UI in your web browser by visiting http://localhost:4646/ui(opens in new tab).


