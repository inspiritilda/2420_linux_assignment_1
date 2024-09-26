# Setting up a DigitalOcean droplet using `doctl` and `cloud-init`

## Table of Contents
- [Setting up a DigitalOcean droplet using `doctl` and `cloud-init`](#setting-up-a-digitalocean-droplet-using-doctl-and-cloud-init)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
    - [Prerequisites:](#prerequisites)
  - [Installing and Setting up doctl](#installing-and-setting-up-doctl)
    - [Steps to Install doctl:](#steps-to-install-doctl)
  - [Generating API token](#generating-api-token)
    - [To generate a personal access token:](#to-generate-a-personal-access-token)
    - [Use the API token to grant account access to doctl](#use-the-api-token-to-grant-account-access-to-doctl)
  - [Authenticate `doctl`:](#authenticate-doctl)
    - [Validate that doctl is working](#validate-that-doctl-is-working)
  - [Setting up a SSH key](#setting-up-a-ssh-key)
    - [Understanding SSH Keys](#understanding-ssh-keys)
  - [Configuring cloud-init](#configuring-cloud-init)
    - [What is Cloud-Init?](#what-is-cloud-init)
    - [Sample cloud-init Configuration File:](#sample-cloud-init-configuration-file)
    - [Breakdown of Each Section:](#breakdown-of-each-section)
  - [Deploying the droplet](#deploying-the-droplet)
    - [Droplet Creation Command:](#droplet-creation-command)
    - [Breakdown of Each Section:](#breakdown-of-each-section-1)
  - [Verifying everything worked](#verifying-everything-worked)
  - [References](#references)

## Introduction
This guide will show you how to create an Arch Linux droplet on DigitalOcean using `doctl` command-line tool and `cloud-init` (DigitalOcean, n.d.).
We will go step by step to set up SSH keys for secure access, upload a custom Arch Linux image, and automate the setup process with `cloud-init`.

### Prerequisites:
- A DigitalOcean account.
- `ssh` installed on your local machine.
- A basic understanding of how to use the Linux command line.
- This guide assumes you already have an Arch Linux droplet and can SSH into it.

## Installing and Setting up doctl
`doctl` is the official DigitalOcean CLI tool, and it makes it super easy to manage everything right from your terminal (DigitalOcean, n.d.).

### Steps to Install doctl:
On Arch Linux, install `doctl` with the pacman package manager. You can run:
```bash
sudo pacman -S doctl
```
![installing doctl](images/installing%20doctl.png)
This installs the `doctl` package on your Arch Linux system using the pacman package manager.

## Generating API token
When you run the authentication, it will ask for an API token. This is the only step that needs to be done outside the Arch Linux droplet.

### To generate a personal access token:
  1. Log-in to your DigitalOcean Control Panel.
  2. On the left menu, click API (this takes you to the "Applications & API" page under the Tokens tab).
  3. In the Personal access tokens section, click the Generate New Token button.
  4. You will receive your own personal access Token jsut like the screenshot below.

![personal access token](images/new%20personal%20token.png)

### Use the API token to grant account access to doctl
Now that you have your API token, you can use it to link `doctl` to your DigitalOcean account. When you run `doctl auth init`, just enter the token when it asks for it. It looks like this:
```bash
doctl auth init
```
![validating token](images/validating%20token.png)

## Authenticate `doctl`:
Once you have installed `doctl`, you will need to link it to your DigitalOcean account. To do that, run:
```bash
doctl auth init
```
![authenticate doctl](images/authenticate%20doctl.png)

### Validate that doctl is working
To confirm that you have successfully linked `doctl` to your account, you can look at your account details by running:
```bash
doctl account get
```
If successful, the output looks like:
```bash
Email                      Droplet Limit    Email Verified    UUID                                        Status
sammy@example.org          10               true              3a56c5e109736b50e823eaebca85708ca0e5087c    active
```
![validate doctl](images/validate%20account%20link.png)

## Setting up a SSH key
SSH keys are great for secure, passwordless access to your server (Arch Linux, n.d.).

### Understanding SSH Keys
SSH (Secure Shell) keys are a pair of cryptographic keys used for authenticating secure connections. Unlike passwords, SSH keys provide a more secure method of authentication as they are not transmitted over the network, making them resistant to brute-force attacks (Sobel, 2020). By using SSH keys, you can log in to your server without the need for a password, enhancing security.

1. **Generate SSH Key Pair:**
    ```bash
    ssh-keygen -t ed25519 -f ~/.ssh/do-key -C "your email address"
    ```
    This command generates a new SSH key pair. Follow the prompts to save it in the default location.

2. **Add SSH Key to DigitalOcean:**
    Once your SSH key is generated, add it to your DigitalOcean account with:
    ```bash
    doctl compute ssh-key create <key-name> --public-key-file ~/.ssh/<your-key>.pub
    ```

## Configuring cloud-init

### What is Cloud-Init?
`cloud-init` is a tool used in cloud environments to automate the initial setup of virtual machine (Canonical, n.d.; DigitalOcean, n.d.). It allows users to configure settings like network configuration, user creation, and package installation during the first boot of the instance (Canonical, n.d.; Cloud-init, n.d.). This automation reduces manual setup time and ensures consistency across deployments.

### Sample cloud-init Configuration File:
After you upload your public SSH key to your DigitalOcean account, create a file named `cloud-config.yml` with the following content:
```bash
#cloud-config
users:
  - name: user-name
    primary_group: group-name
    groups: wheel
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-ed25519 ...

packages:
  - ripgrep
  - rsync
  - neovim
  - fd
  - less
  - man-db
  - bash-completion
  - tmux

disable_root: true
```

### Breakdown of Each Section:
* `users`: This section specifies the user accounts to be created on the droplet.
* `- name: user-name`: Creates a user account with the specific username(`user-name`). This comment indicates that this should be replaced with the desired username.
* `primary_group: group-name`: Assigns the specified primary group to the user. This should be changed to the desired group name. The primary group determines the default group the user belongs to.
* `groups: wheel`: Adds the user to the `wheel` group, which is typically used in Unix-like systems to grant users the ability to use the `sudo` command. This allows the user to perfrom administrative tasks.
* `shell: /bin/bash`: Sets the user's default shell to `bash`, which is a common command-line interface in Linux environments.
* `sudo: ['ALL=(ALL) NOPASSWD:ALL']`: Configures the user to run any command with `sudo` without being prompted for a password. This can simplify automated scripts and commands that require administrative privileges.
* `ssh-authorized-keys:`: This section allows you to specify SSH public keys that are authorized for the user to log in without a password.
* `- ssh-ed25519 ...`: Specifies the SSH public key to be added for this user. The `ssh-ed25519` is a type of SSH key. The ellipsis (`...`) should be replaced with the actual public key.
* `packages:`: This section lists the packages that should be installed on the droplet during its initialization.
  - `- ripgrep`: A fast search tool that recursively searches your current directory for a regex pattern.
  - `- rsync`: A utility for efficiently transferring and synchronizing files across systems.
  - `- neovim`: An extensible text editor that serves as an improvement to the popular Vim editor.
  - `- fd`: A simple, fast, and user-friendly alternative to `find`, which helps locate files and directories.
  - `- less`: A terminal pager program that allows you to view (but not change) the contents of files one screen at a time.
  - `- man-db`: The manual page database for Linux, allowing you to view documentation about commands and programs using the `man` command.
  - `- bash-completion`: A package that provides command-line auto-completion for Bash, making it easier to work with commands and options.
  - `- tmux`: A terminal multiplexer that allows you to create and manage multiple terminal sessions within a single window.
* `disable_root: true`: Disables the root user account for security reasons, preventing direct root login and encouraing the use of the `sudo` command for adminitrative tasks instead.

## Deploying the droplet
Now that everything is configured, you're ready to deploy your Arch Linux droplet using `doctl` and the `cloud-init` configuration file.

### Droplet Creation Command:
To create the droplets, run the following command:
``` bash
doctl compute droplet create --image arch-linux-2024-01-01 --size s-1vcpu-1gb --region nyc1 --ssh-keys git-user --user-data-file <path-to-your-cloud-init-file> --wait first-droplet second-droplet
```
### Breakdown of Each Section:
* `--image arch-linux-2024-01-01`: This option specifies the operating system image for the droplets, which in this case is arch-linux-2024-01-01.
* `--size s-1vcpu-1gb`: This specifies the resources for each droplet. Here, each droplet is allocated one virtual CPU and 1 GB of RAM.
* `--region nyc1`: This option defines the region for deploying the droplets. In this example, the droplets will be created in the NYC1 datacenter.
* `--ssh-keys`: This parameter allows you to import SSH keys into the droplet from your DigitalOcean account. You can list available keys using the command `doctl compute ssh-key list`.
* `--user-data-file <path-to-your-cloud-init-file`: This specifies the path to your `cloud-config.yaml file`. For instance, it could look something like `/Users/example-user/cloud-config.yaml`.
* `--wait`: This ensures that the command will not return until the droplet is fully created and ready for use.
* `first-droplet second-droplet`: These are the names of the droplets being created. You can name as many droplets as you want by adding more names at the end of the command.

After executing the command, the terminal will not display a prompt until the droplets finish deploying, which may take a few minutes. Once the deployment is successful, the output will resemble the following:
```bash
ID           Name              Public IPv4       Private IPv4    Public IPv6    Memory    VCPUs    Disk    Region    Image               VPC UUID                                Status    Tags    Features                            Volumes
311143987    second-droplet    203.0.113.199    203.0.113.4                     1024      1        25      nyc1      Ubuntu 22.04 x64    cfcbcc95-365a-4705-a18d-54abde1fc7b4    active            droplet_agent,private_networking
311912986    first-droplet     203.0.113.146    203.0.113.3                     1024      1        25      nyc1      Ubuntu 22.04 x64    cfcbcc95-365a-4705-a18d-54abde1fc7b4    active            droplet_agent,private_networking
```

## Verifying everything worked
After successfully deploying the droplets, it is important to check if the cloud-init configuration was executed properly. You can log in to one of the droplets using the username defined in the `cloud-config.yaml file`. To do this, you can use the OpenSSH command shown below. Make sure to replace the placeholder with the public IPv4 address of your droplet:
```bash
ssh example-user@<your-droplet-ip-address>
```
If everything is set up correctly, your terminal prompt should change to something like this:
```bash
example-user@first-droplet:~$
```
At this point, you are logged in and can explore the droplet.

To verify that the nginx configuration is functioning, simply copy one of the droplet's public IP addresses and paste it into your web browser, then press Enter. You should see the nginx homepage, which will display the name of your droplet along with its IP address. This indicates that your setup was successful.

## References
- Arch Linux. (n.d.). *SSH Keys*. Retrieved from [https://wiki.archlinux.org/title/Secure_Shell#SSH_keys](https://wiki.archlinux.org/title/Secure_Shell#SSH_keys)

- Canonical. (n.d.). *Cloud-init*. Retrieved from [https://cloud-init.readthedocs.io/en/latest/](https://cloud-init.readthedocs.io/en/latest/)

- Cloud-init. (n.d.). *cloud-init docs*. Retrieved from [https://docs.cloud-init.io/en/latest/index.html](https://docs.cloud-init.io/en/latest/index.html)

- Sobel, M. (2020). *Learning Linux for iOS development: An introduction to Unix-based development for mobile applications*. Packt Publishing.
