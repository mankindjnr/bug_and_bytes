# install docker and docker-compose in a vm

Step 1: Update Your Package List
First, update your package list to ensure you have the latest information about the available packages:

```bash
sudo apt update
Step 2: Install Docker
If Docker is not already installed on your droplet, you can install it with the following commands:
```

```bash
sudo apt install -y docker.io
Start Docker and enable it to start on boot:
```

```bash
sudo systemctl start docker
sudo systemctl enable docker
Step 3: Install Docker Compose
Check the Docker Compose releases page for the latest version and replace 1.29.2 in the command below with the latest version if it's different.
```

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
Step 4: Apply Executable Permissions to the Binary
Make the Docker Compose binary executable:
```

```bash
sudo chmod +x /usr/local/bin/docker-compose
Step 5: Verify the Installation
Verify that Docker Compose is installed correctly by checking the version:
```

```bash
docker-compose --version
Optional Step: Enable Non-root User to Run Docker Commands
If you want to run Docker commands without using sudo, add your user to the docker group:
```

```bash
sudo usermod -aG docker $USER
After running the above command, log out and log back in for the changes to take effect. Alternatively, you can use newgrp to apply the changes immediately:
```

```bash
newgrp docker
Summary of Commands
Here’s a summary of all the commands you need to run:
```

```bash
# Update package list
sudo apt update

# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify Docker Compose installation
docker-compose --version

# (Optional) Enable non-root user to run Docker commands
sudo usermod -aG docker $USER
newgrp docker
```

By following these steps, Docker Compose will be installed and ready to use on your DigitalOcean droplet.

##install git in the vm

Step 1: Update Your Package List
First, ensure your package list is up-to-date:

```bash
sudo apt update
Step 2: Install Git
Install Git using the package manager:
```

```bash
sudo apt install git -y
Step 3: Verify the Installation
Verify that Git has been installed correctly by checking the version:
```

```bash
git --version
Configuring Git (Optional)
After installing Git, it's a good practice to set your user name and email. This information will be used in the commits you create.

Set your Git user name:
```

```bash
git config --global user.name "Your Name"
Set your Git email:
```

```bash
git config --global user.email "your.email@example.com"
Summary of Commands
Here’s a summary of all the commands you need to run:
```

```bash
# Update package list
sudo apt update

# Install Git
sudo apt install git -y

# Verify Git installation
git --version

# Optional: Configure Git user name and email
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
By following these steps, Git will be installed and configured on your DigitalOcean droplet or any other VM running Ubuntu.
```
