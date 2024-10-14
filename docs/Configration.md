# **Create VMs**

## **Provisioning VMs for the Project**

The following VMs are needed for the project:

* **1- Ubuntu VM for K8S**: This VM will be used as the Kubernetes master node.
* **2- Ubuntu VM for Jenkins**: This VM will be used as the Jenkins server.
* **3- Ubuntu VM for SonarQube**: This VM will be used as the SonarQube server.
* **4- Ubuntu VM for Nexus**: This VM will be used as the Nexus server.
## **Install and setup K8S** 
 
### 1. Install Docker[On Master & Worker Node]
```bash
sudo apt install docker.io -y
sudo chmod 666 /var/run/docker.sock
```

### 2. Install Required Dependencies for Kubernetes[On Master & Worker Node]    

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings
```
    
### 3. Add Kubernetes Repository and GPG Key[On Master & Worker Node]
 
```bash
 curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
 ### 4. Update Package List[On Master & Worker Node]
```bash
sudo apt update
```
 
### 5. Install Kubernetes Components[On Master & Worker Node]
```bash
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1`
``` 
### 6. Initialize Kubernetes Master Node [On MasterNode]
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
``` 
### 7. Configure Kubernetes Cluster [On MasterNode]
```bash    
        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

 
## 1.2 Installing Jenkins on Ubuntu

```bash
#!/bin/bash
# Install OpenJDK 17 JRE Headless
sudo apt install openjdk-17-jre-headless -y

# Download Jenkins GPG key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

#Add Jenkins repository to package manager sources
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

```
### Update package manager repositories

```bash
sudo apt-get update
```

### Install Jenkins
```bash
sudo apt-get install jenkins -y
```

- **Save this script in a file, for example, install_jenkins.sh, and make it executable using**
```bash  
chmod +x install_jenkins.sh
./install_jenkins.sh
```

## **Install docker for furture use** 
```bash
#!/bin/bash
#Update package manager repositories
sudo apt-get update

#Install necessary dependencies
sudo apt-get install -y ca-certificates curl

#Create directory for Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings

#Download Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc`

#Ensure proper permissions for the key
sudo chmod a+r /etc/apt/keyrings/docker.asc

#Add Docker repository to Apt sources
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
#Update package manager repositories
`sudo apt-get update`

#Install Docker components
```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Save this script in a file, for example, `install_docker.sh`, and make it executable using.
   
```bash
chmod +x install_docker.sh
```

### Then, you can run the script using.

```bash
./install_docker.sh
```


# Create Sonarqube Docker container

## To run SonarQube in a Docker container with the provided command, you can follow these steps:
### 1. log in to your Docker Hub account from your Docker host, you can use the following command: 
```bash
    docker login
```

### 2. Enter your Docker Hub username and password when prompted

 `Username: your_dockerhub_username` 
 `Password: your_dockerhub_password`

- It will print `Login Succeeded`

### 2. Run the following command:
```bash
docker run -d --name sonar --network host sonarqube:lts-community
```    
**Note: i use ( -- network host ) to can abel to access my image from host machine**

- This command will download the sonarqube:lts-community Docker image from Docker Hub if it's not already  available locally. Then, it will create a container named "sonar" from this image, running it in detached mode (-d flag) and mapping port 9000 on the host machine to port 9000 in the container (<ip-addrees-for-this-machine:9000> ).

# Install and SetUp Nexus 
```bash
    #!/bin/bash
    # Update package manager repositories
    sudo apt-get update
    # Install necessary dependencies
    sudo apt-get install -y ca-certificates curl
    # Create directory for Docker GPG key
    sudo install -m 0755 -d /etc/apt/keyrings
    # Download Docker's GPG key
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    # Ensure proper permissions for the key
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    # Add Docker repository to Apt sources
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    # Update package manager repositories
    sudo apt-get update
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Save this script in a file, for example, install_docker.sh, and make it executable using:
```bash
    chmod +x install_docker.sh
```
### Then, you can run the script using:
```bash
    ./install_docker.sh 
```
## Create Nexus container


### To create a Docker container named "nexus" on port  and exposing it on port 8081, you can use the following command: 
```bash
    docker run -d --name nexus -network host  sonatype/nexus3:latest
```    
### After running this command, Nexus will be accessible on your host machine at http://IP:8081.

## Get Nexus default password
- Your provided commands are correct for accessing the Nexus password stored in the container. Here's a breakdown of the steps:

### Get Container ID
```bash
docker ps
```
### Access Container's Bash Shell: Once you have the container ID, you can execute the docker exec command to access the container's bash shell:
```bash
docker exec -it <container_ID> /bin/bash
```
### Navigate to Nexus Directory:
```bash 
cd sonatype-work/nexus3
```        
### View Admin Password: Finally, you can view the admin password by displaying the contents of the admin.password file:
```bash
cat admin.password
```
### Exit the Container Shell:
```bash   
exit
```
 This process allows you to access the Nexus admin password stored within the container. Make sure to keep this password secure, as it grants administrative access to your Nexus instance.

