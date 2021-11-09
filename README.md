
# Install and Configure a complete Ubuntu Server on a Raspberry Pi 4

Guidelines for install and configure a full Ubuntu Server on a Raspberry Pi 4 system.

### Table of Contents

- [Ubuntu Server](#ubuntu-server)
    - [Ubuntu server installation](#ubuntu-server-installation)
    - [First SSH access](#first-ssh-access)
    - [SSH security - PubkeyAuthentication](#ssh-security-pubkeyauthentication)
- [Docker](#docker)
    - [Docker installation](#docker-installation)
    - [Configuration and Testing](#configuration-and-testing)
    - [Useful Docker commands](#useful-docker-commands)
- [IPsec VPN Server on Docker](#ipsec-vpn-server-on-docker)
    - [Configure the VPN details](#configure-the-vpn-details)
    - [Start the IPsec VPN server](#start-the-ipsec-vpn-server)
    - [Check the VPN server status](#check-the-vpn-server-status)
    - [Retrieve VPN login details](#retrieve-vpn-login-details)
    - [Connect to IPsec VPN](#connect-to-ipsec-vpn)

- [References](#references)

## Ubuntu Server

### Ubuntu server installation

First, prepare your Raspberry Pi downloading and installing Ubuntu Server on your sd card with
<a href="https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#2-prepare-the-sd-card">Rpi Imager </a>

After the installation is completed, connect your Pi to your router with an ethernet cable and boot it.

### First SSH access

Retrive your IP server address from your HomeGateway interface and then open an ssh connection to your server.

```bash
# 'ubuntu' is the default username generated from cloud-init configuration 
ssh ubuntu@[your_ip_server_address]
```
so something like this:

```bash
ssh ubuntu@192.168.1.150
```

You will be asked to confirm the connection:

```bash
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type “yes” to confirm.

The system asks you to put the user password.

```bash
# 'ubuntu' is the default password
ubuntu@[your_ip_server_address]s password: 'ubuntu'
```
Complete the configuration changing the password.

### SSH security - PubkeyAuthentication

Open a terminal on the host and generate a new SSH keypair with the ssh-keygen command:

 ```bash
$ ssh-keygen -b 4096 -C my_key
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:iWpy+YnHAaQd98yfVMO45k2y0GZfouugBArd5K8RiHg my_key
The key's randomart image is:
+---[RSA 4096]----+
|           o     |
|    o .   . +    |
|   +.o + . o .   |
|.o.=o  .=.O o .  |
|+ E =.. SO O o   |
| o . *.   * o    |
|  o *.o..  .     |
|   + *oo ..      |
|    o.+  ..      |
+----[SHA256]-----+
```
**Note:**
 If you have already generated an SSH keypair that you would like to use, skip this bash command

When your keys are generated, you can copy the public key to the server. You can use script ssh-copy-id or just copy/paste the content of the id_rsa.pub.
 
 ```bash
$ ssh-copy-id [your_ip_server_address]
```
or in Windows :
 ```bash
type C:\\Users\[username]\.ssh\id_rsa.pub | ssh ubuntu@[your_ip_server_address] "cat >> .ssh/authorized_keys"
```

Now try logging into the machine, with:   "ssh 'ubuntu@192.168.1.150' "
and check to make sure that only the key(s) you wanted were added.


## Docker

Before installing Docker, you should update all packages on your Raspberry Pi OS. To do so, first update the APT package repository cache and then all the packages with the following commands:


 ```bash
$ sudo apt update
$ sudo apt upgrade -y

#after package update do a reboot
$ sudo reboot
```

### Docker installation

To install Docker on your Raspberry Pi OS, you must download the Docker installation script on your Raspberry Pi 4 and run it.

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo bash get-docker.sh
```
At this point, Docker should be installed.

[![docker-installation](https://github.com/teodoro98/Pi4Server/blob/main/src/docker-installation.JPG?raw=true)](https://github.com/teodoro98/Pi4Server/tree/main/src/)

### Configuration and Testing

Once Docker is installed, add your login user to the docker group with the following command:

```bash
$ sudo usermod -aG docker $(whoami)

#for the changes to take effect, reboot your system
$ sudo reboot
```

Verify that Docker is installed correctly by running the hello-world image.

```bash
 $ docker run hello-world
```

You should see a result like this:

[![docker-hello-world](https://github.com/teodoro98/Pi4Server/blob/main/src/docker-hello-world.JPG?raw=true)](https://github.com/teodoro98/Pi4Server/tree/main/src/)

### Useful Docker commands

You can list all the running Docker containers with the following command:

```bash
$ docker container ls
```
[![docker-container-ls](https://github.com/teodoro98/Pi4Server/blob/main/src/docker-container-ls.JPG?raw=true)](https://github.com/teodoro98/Pi4Server/tree/main/src/)

You can name a Docker container with the –name command line argument.

```bash
$ docker run -d -p 8081:80 --name webserver2 httpd
```

You can stop a running Docker container using the name or the ID of the running container.

```bash
$ docker container stop webserver2
#or
$ docker container stop [CONTAINER ID]
```

You can remove a stopped Docker container using the name or the ID of the running container.

```bash
$ docker container rm webserver2
#or
$ docker container rm [CONTAINER ID]
```


## IPsec VPN Server on Docker

First make sure that you have successfully completed the step [Docker](#docker).

### Configure the VPN details

You have to download the [vpn.env](https://github.com/teodoro98/Pi4Server/blob/main/src/vpn.env) file needed for the VPN configuration.

```bash
$ wget https://github.com/teodoro98/Pi4Server/blob/main/src/vpn.env
```
Modify it with an editor like 'nano' or 'vim', uncomment and replace the field:
- VPN_IPSEC_PSK (pre-shared key, you can create one with  ` $ openssl rand -base64 16`)
- VPN_USER (username of default vpn client)
- VPN_PASSWORD (password of default vpn client)

### Start the IPsec VPN server

Create a new Docker container from the IPsec VPN server image (customaized with you env.vpn configuration file)

```bash
docker run \
    --name ipsec-vpn-server \
    --env-file vpn.env \
    --restart=always \
    -v ikev2-vpn-data:/etc/ipsec.d \
    -p 500:500/udp \
    -p 4500:4500/udp \
    -d --privileged \
    hwdsl2/ipsec-vpn-server
```

### Check the VPN server status

Check the status of the IPsec VPN server:

```bash
$ docker exec -it ipsec-vpn-server ipsec status
```

Show currently established VPN connections:

```bash
$ docker exec -it ipsec-vpn-server ipsec trafficstatus
```

### Retrieve VPN login details

You can retrive you VPN login credentials viewing the container logs:

```bash
docker logs ipsec-vpn-server
```

Search for these lines in the output:

```bash
Connect to your new VPN with these details:

Server IP: your_vpn_server_ip
IPsec PSK: your_ipsec_pre_shared_key
Username: your_vpn_username
Password: your_vpn_password
```

### Connect to IPsec VPN

You can now connect to the VPN selecting L2TP/IPSec PSK like type method, and putting the right parameteres showed before in [Retrieve VPN login details](#retrieve-vpn-login-details)


## References 

Link used for reference
    
### Ubuntu Server
    - https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview
    - https://steemit.com/utopian-io/@jamzed/9-things-i-do-after-installing-a-fresh-linux-server-ubuntu 
### Docker
    - https://docs.docker.com/engine/install/ubuntu/
    - https://linuxhint.com/install_docker_raspberry_pi-2/
### IPsec VPN
    - https://github.com/hwdsl2/docker-ipsec-vpn-server
