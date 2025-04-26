# Terraform Installation on Linux

Follow these steps to install Terraform on a Linux system:

* Step 1: Update the Package List and install gnupg,software-properties-common
```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
```

* Step 2: Install the HashiCorp GPG key
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

```

* Step 3: Verify the key's fingerprint.
```bash
gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint

```

* Step 4: lsb_release -cs command finds the distribution release codename for your current system
```bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

```

* Step 5: Download the package information from HashiCorp.
```bash
sudo apt update
```

* Step 6: sudo apt-get install terraform
```bash
sudo apt-get install terraform
```

Terraform is now installed and ready to use!

* Create a directory and file main.tf
```bash
mkdir terraform-docker && cd terraform-docker && touch main.ft
```
* content of main.ft file to provision two linux containers and enable sftp on one of the servers.

```json
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.4.0"
    }
  }
}

provider "docker" {}

# --- Create a custom bridge network ---
resource "docker_network" "alpine_net" {
  name = "alpine-network"
}

# --- Container 1: Alpine with user 'rainun' ---
resource "docker_container" "alpine_1" {
  name  = "alpine-container-1"
  image = "alpine:latest"
  # Keep container running indefinitely
  command = ["tail", "-f", "/dev/null"]

  # *** Attach to the custom network ***
  networks_advanced {
    name = docker_network.alpine_net.name
  }

  provisioner "local-exec" {
    command = <<-EOF
      echo "Waiting for container ${self.name} to be ready..."
      sleep 5
      echo "Configuring user 'rainun' in ${self.name}..."
      docker exec ${self.name} sh -c ' \
        apk update && apk add --no-cache shadow && apk add openssh-client &&\
        useradd -m -s /bin/ash rainun && \
        echo "rainun:rainun" | chpasswd \
      '
      echo "User 'rainun' configured in ${self.name}."
    EOF
    when = create
  }

  # Ensure container depends on the network being created
  depends_on = [docker_network.alpine_net]
}

# --- Container 2: Alpine with user 'rainun' AND SFTP Enabled ---
resource "docker_container" "alpine_2_sftp" {
  name  = "alpine-container-2-sftp"
  image = "alpine:latest"
  command = ["tail", "-f", "/dev/null"]

  # *** Attach to the custom network ***
  networks_advanced {
    name = docker_network.alpine_net.name
  }

  ports {
    internal = 22
    external = 2222 # Host mapping remains the same
  }

  # Provisioner 1: Create user
  provisioner "local-exec" {
    command = <<-EOF
      echo "Waiting for container ${self.name} to be ready..."
      sleep 5
      echo "Configuring user 'rainun' in ${self.name}..."
      docker exec ${self.name} sh -c ' \
        apk update && apk add --no-cache shadow && \
        useradd -m -s /bin/ash rainun && \
        echo "rainun:rainun" | chpasswd \
      '
      echo "User 'rainun' configured in ${self.name}."
    EOF
    when = create
  }

  # Provisioner 2: Install OpenSSH, configure SFTP, and start SSHD
  provisioner "local-exec" {
    command = <<-EOF
      echo "Configuring SSH/SFTP in ${self.name}..."
      docker exec ${self.name} sh -c ' \
        apk update && apk add --no-cache openssh && \
        ssh-keygen -A && \
        echo "Configuring sshd_config..." && \
        sed -i \
            -e "s/^#?PermitRootLogin.*/PermitRootLogin no/" \
            -e "s/^#?PasswordAuthentication.*/PasswordAuthentication yes/" \
            -e "s/^#?Subsystem\\s*sftp.*/Subsystem sftp /usr/lib/ssh/sftp-server/" \
            /etc/ssh/sshd_config && \
        grep -qxF "AllowUsers rainun" /etc/ssh/sshd_config || echo "AllowUsers rainun" >> /etc/ssh/sshd_config && \
        echo "Starting sshd..." && \
        /usr/sbin/sshd \
      '
      echo "SSH/SFTP configured and started in ${self.name}."
      echo "--> SFTP access from host: sftp -P 2222 rainun@localhost"
    EOF
    when = create
  }

  # Ensure container depends on the network being created
  depends_on = [docker_network.alpine_net]
}
```

* Terraform initialize 
Initializing a configuration directory downloads and installs the providers defined in the configuration

```bash
terraform init
```

* Validate and format the content of main.ft file
```bash
terraform fmt
terraform validate
```