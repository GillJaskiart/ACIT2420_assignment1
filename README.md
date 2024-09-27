# ACIT2420_assignment1
Assignment 1 - ACIT 2420 - Linux System Administration

Name: Jaskirat Gill
Student ID: A01349758
## Introduction and general information
### What is SSH?
SSH (Secure Shell) is a cryptographic network protocol used to securely access and manage remote devices over a network. It provides secure encrypted communications between two untrusted hosts, often employed for remote login, file transfers, and network services, replacing older, less secure protocols like Telnet and FTP. SSH ensures data integrity and confidentiality over unsecured networks such as the internet.

Ylonen, T., & Lonvick, C. (2006). **The Secure Shell (SSH) protocol architecture**. IETF. https://datatracker.ietf.org/doc/html/rfc4251 
### What is `Doctl`?
**doctl** is a command-line interface (CLI) tool used to interact with DigitalOcean’s API. It allows users to manage their DigitalOcean cloud resources, such as creating and managing Droplets (virtual machines), networking, and storage, without needing to use the DigitalOcean web interface. The tool enables automation and easier integration with scripts and CI/CD pipelines, facilitating cloud infrastructure management for developers and sysadmins.

DigitalOcean. (n.d.). **doctl command line tool**. DigitalOcean Documentation. https://docs.digitalocean.com/reference/doctl/
### What is a `cloud-init` file?
**cloud-init** is a widely used tool for initializing cloud instances at boot time. It allows the configuration of a virtual machine or cloud instance during its first boot by passing user data, such as scripts or configuration files. cloud-init can set up user accounts, configure networking, install software, and perform other instance customizations, making it crucial for automating cloud infrastructure.

Canonical Ltd. (n.d.). **Cloud-init: The standard for cloud instance initialization**. https://cloud-init.io/

## Recap
- You already have a DigitalOcean account and an Arch Linux droplet that you created using the website. 
- While making the droplet, you uploaded and used a custom image for Arch Linux.
- If you do not have the droplet yet, follow the instructions below to create one.
### Create a custom Arch Linux droplet on DigitalOcean
Before we begin our `Doctl` and `cloud-config` setup, here are the steps to quickly create an Arch Linux droplet on DigitalOcean:
1. Go to https://www.digitalocean.com and create an account.
2. In the left navigation panel, go to **Manage > Backups & Snapshots**, then click on **Custom Images** to upload the Arch Linux image. You can download the image by clicking on this link: https://gitlab.archlinux.org/archlinux/arch-boxes/-/package_files/7529/download 
3.  In the left navigation panel, go to **Settings**, then click **Security**. Click the button **Add SSH Key**. Paste the content of your public key and give a name to your key. ![[Screenshot 2024-09-27 at 2.25.38 PM.png]] Then click **Add SSH Key**.
4. Click the **Create** button in the top to start creating a droplet.
5. Select **San Francisco - Datacenter 3** (SFO3) as the region as it is closest to Vancouver. 
6. In the Choose an image section, click on **Custom Images** and select the Arch Linux image that you uploaded earlier.
7. Choose the **Basic plan** with Premium AMD, 1 GB CPU and 25 GB SSD.
8. Select the SSH key you created in step 3. ![[Screenshot 2024-09-27 at 2.29.22 PM.png]]
9. Click **Create Droplet**. You now have your first droplet.
## Objective
In this tutorial, we will:
- SSH into the existing droplet and connect to your DigitalOcean account through CLI.
- Create new SSH key pair and a `cloud-config` file.
- (Optional) Add a custom Arch Linux image using `doctl`.
- Create a new droplet using the SSH key, `cloud-config` and the custom image file. 
## Step 1: SSH into your existing droplet
Open your terminal and SSH into your existing droplet using the command `ssh <your hostname>`. ![[Screenshot 2024-09-27 at 2.37.09 PM.png]]
#### Trouble-shooting for `ssh` command
If you are unable to use this command, create a config file in .ssh folder if you don't have it already. The content of config file is
```
Host arch
  HostName 143.198.140.15
  User arch
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/do-key
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```

## Step 2: Create SSH key pair
When you SSH into your droplet, you will be in the home directory by default, if you are not in home directory, use this command to go to home directory:
```terminal
cd ~
```

Use `ls -la` command to check if you have `.ssh` directory. If you do not have this directory, make it using the command: 
```terminal
mkdir ~/.ssh
```

Now we will generate a pair of SSH key (public and private key) using the command:
```bash
ssh-keygen -t ed25519 -f ~/.ssh/do-key -C "your email address"
```
Make the adjustments to the **email address** before running the command.

It will ask for a **passphrase**, you can press enter to skip this part (If you do use a passphrase, type something that you will remember).

If you are able to create your key pair successfully, you will see an output like this. 
![[Screenshot 2024-09-27 at 2.46.26 PM.png]]

You will now have two plain text files `do-key` and `do-key.pub` in your `.ssh` folder. We will add the public key to the DigitalOcean account later. 
## Step 3: Install `Doctl`
To install `Doctl`, you need to first update your package database and upgrade all system packages to their latest versions using the command `sudo pacman -Syu`.

After you have updated, you can install `Doctl` using the command:
```bash
sudo pacman -S doctl
```

You should see an output like this.

![[Screenshot 2024-09-27 at 3.11.13 PM.png]]
## Step 4: Create API token and authenticate it
To Create API token, go to DigitalOcean website. In the left navigation panel, click on **API**, then click **Generate New Token**. 

Type a name for your token, grant it **full access** and click **Generate Token**. Copy the token as it will be used while authenticating via `doctl`.

To authenticate, go to your droplet (terminal with the SSH connection to your droplet) and run this command.
```terminal
doctl auth init
```
You will be asked for the token. Paste the token and you will get connected to you DigitalOcean account.

You should see this output upon successful completion of this step.
![[Screenshot 2024-09-27 at 3.28.53 PM.png]]

To confirm that you are authorized, run this command to get your account information.
```bash
doctl account get
```
This command will display information about your account. It should look like this.
![[Screenshot 2024-09-27 at 3.32.12 PM.png]]

You can now use `doctl` to communicate directly with DigitalOcean. You can create custom images, droplets and much more by only using the Command Line interface (CLI).
## Step 4: (Optional) Create custom Arch Linux image and add to to the DigitalOcean account
You already have a custom Arch Linux image in you account (the one you used to make your first droplet). But you can also create our own **Arch Linux** image using `doctl`. 

Use this command to create the image.
```bash
doctl compute image create "Arch Linux" --image-url "https://gitlab.archlinux.org/archlinux/arch-boxes/-/package_files/7529/download" --region sfo3 --image-distribution "Arch Linux"
```
In this command, 
- We named out image "Arch Linux" 
- Used the url  "https://gitlab.archlinux.org/archlinux/arch-boxes/-/package_files/7529/download" to get the image from. 
- The `sfo` region refers to San Fransisco Datacenter 3. 
- We  specified the name of the distribution "Arch Linux"

After running this command, you should see this output:
![[Screenshot 2024-09-27 at 3.55.00 PM.png]]
Note that, 
- We get an **ID** for this image. This ID will be used to create a new droplet later.

## Step 5: Upload SSH public key to DigitalOcean account


## Step 6: Make `cloud-init.yml` file
## Step 7: Create the droplet using `Doctl` and `cloud-config`

