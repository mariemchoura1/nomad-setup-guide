# nomad-setup-guide
Guide for installing Nomad, creating a local cluster, and deploying/updating an example application.
## 1. Install Nomad

1. **Install the required packages:**
   $ sudo apt-get update && \
   $ sudo apt-get install wget gpg coreutils
2. **Add the HashiCorp GPG key:**
   $ wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
3. **Add the official HashiCorp Linux repository:**
   $ echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
 4. **Update and install Nomad:**
    $ sudo apt-get update && sudo apt-get install nomad
5. **Verify the installation:**
   $ nomad -v

## 2. Clone the code repository
The example repository includes Terraform configurations for starting a cluster on the cloud as well as the jobspec files for the example application.

1. **Clone the code repository:**
$ git clone https://github.com/hashicorp-education/learn-nomad-getting-started.git

2. **Navigate to the repository folder:**
$ cd learn-nomad-getting-started

3. **Check out the v1.1 tag of the repository as a local branch named nomad-getting-started:**
$ git checkout -b nomad-getting-started v1.1

## 3. Create a cluster
**Prerequisites**
Docker Desktop installed locally
Nomad supports many [task drivers](https://developer.hashicorp.com/nomad/docs/drivers) but the example application in this tutorial uses Docker. Docker Desktop provides the Docker engine, the CLI client, and a graphical interface. Other third-party options are available for running Docker on your machine but we recommend Docker Desktop for the simplicity and ease of use.

Nomad uses a socket file to communicate with Docker and [by default](https://developer.hashicorp.com/nomad/docs/drivers/docker#endpoint), this file is "/var/run/docker.sock" on Linux, Mac, and other Unix platforms, and "/./pipe/docker_engine" on Windows. Docker Desktop creates a symlink to this socket during installation.

If the Nomad client fails to detect the Docker driver with the error Constraint "missing drivers": 1 nodes excluded by filter, the issue may be this missing symlink. Additional information about this symlink for Mac and [Windows](https://docs.docker.com/desktop/windows/permission-requirements/) is available.

Open your terminal and start the development agent. This creates a Nomad cluster of one node that acts as both the server and client.

The -bind flag set to 0.0.0.0 instructs Nomad to listen on all network interfaces present on the machine. The -network-interface flag instructs Nomad to use a network interface other than the default loopback interface (localhost). In this example, Nomad automatically gets the network interface attached to the default route on the machine with the GetDefaultInterfaces function but you can also provide the name of an interface. This flag is necessary for the example application as each service runs in a different container and cannot reach the other services on the loopback address.

1. **Leave Nomad running in this terminal session:**
$ sudo nomad agent -dev \
  -bind 0.0.0.0 \
  -network-interface='{{ GetDefaultInterfaces | attr "name" }}'
  
2. **In a second terminal session, export the cluster address:**
$ export NOMAD_ADDR=http://localhost:4646

## 4. Verify connectivity
$ nomad node status

## 5. Browse the web UI
Navigate to the Nomad UI in your web browser by visiting http://localhost:4646/ui(opens in new tab).

## 6. [Deploy and update a job](https://developer.hashicorp.com/nomad/tutorials/get-started/gs-deploy-job)
you will deploy and update an example application. In the process, you will learn about the Nomad job specification.

The example application runs in Docker containers and consists of a database and a web frontend that reads from the database. You will set up the database with a parameterized batch job and then use a periodic batch job to start additional short-lived jobs that write data to the database.

1. **Navigate to the jobs directory of the example repository on your local machine:**
$ cd jobs


Each of the jobspec files below that make up the application sets the driver attribute to docker and specifies an image stored on the GitHub Container Registry in the config block with the image attribute. The Redis job is the exception as it uses an official Redis image hosted on Docker Hub. By default, Nomad looks for images on Docker Hub so the full path including https:// is not necessary for the Redis job.

**pytechco-redis.nomad.hcl** - This service job runs and exposes a Redis database as a Nomad service for the other application components to connect to. The jobspec sets the type to service, configures a Nomad service block to use Nomad's native service discovery, and creates a service named redis-svc.

**pytechco-web.nomad.hcl** - This service jobs runs the web application frontend that displays the values stored in the database and the active employees. The jobspec sets the type to service and uses a static port of 5000 for the application. It also uses the nomadService built-in function to retrieve address and port information of the Redis database service.

**pytechco-setup.nomad.hcl** - This parameterized batch job sets up the database with default values. You can dispatch it multiple times to reset the database. The jobspec sets the type to batch and has a parameterized block with a meta_required attribute that requires a value for budget when dispatching.

**pytechco-employee.nomad.hcl** - This periodic batch job brings an employee online. It randomizes the employee's job type and other variables such as how long they work for and the rate at which they complete their tasks. The jobspec sets the type to batch and has a periodic block that sets the cron attribute to a value that will allow it to start a new job every 3 seconds.

2. **Deploy the application** Submit the database job:
$ nomad job run pytechco-redis.nomad.hcl

Submit the webapp frontend job:
$ nomad job run pytechco-web.nomad.hcl

Run the command to get the IP address of the node where the job is running:
$ nomad node status -verbose \
    $(nomad job allocs pytechco-web | grep -i running | awk '{print $2}') | \
    grep -i ip-address | awk -F "=" '{print $2}' | xargs | \
    awk '{print "http://"$1":5000"}'

Submit the setup job:
$ nomad job run pytechco-setup.nomad.hcl

Dispatch the setup job by providing a value for budget:
$ nomad job dispatch -meta budget="200" pytechco-setup

Submit the employee job:
$ nomad job run pytechco-employee.nomad.hcl

Navigate to the Nomad UI, click on the Jobs page, and then click on the pytechco-employee job. Since this is a cron batch job, you can see that it creates a new job every three seconds.
