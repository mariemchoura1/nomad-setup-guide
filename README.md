# nomad-setup-guide
Guide for installing Nomad, creating a local cluster, and deploying an example application.
## 1. Install Nomad
Install the required packages.
1. **Install** the required packages.
   sudo apt-get update && \
   sudo apt-get install wget gpg coreutils
2. **Add the HashiCorp GPG key**
   wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
3. **Add the official HashiCorp Linux repository**
   echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
 4. **Update and install Nomad**
    sudo apt-get update && sudo apt-get install nomad

