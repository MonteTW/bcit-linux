
# The Purpose for This Instruction
Since you already created an account and an arch droplet. This instruction is to help you using the main techniques below to generate droplets through your terminal:
1. DigitalOcean API token
2. `doctl` package
3. SSH keys
4. Cloud-init yaml file
5. config file to connect to your droplet easily
   
# Create an API Token[^3] [^4]
What is API token and why?
API token is a string of key to help DigitalOcean authenticate your device and allow the device to get the authority to control your account.

1. Login to Digitalocean control panel.
2. Click API in the left menu.
3. Click **Generate New Token** in the Personal access tokens section.
4. Type in the name of the token and select 90 days expiration.
5. Choose **Full Access** for the Scopes.
   Because this is for your own usage, and we already set the time limitation, it reduces the possibility of leaking the token to someone else after this course.
6. Click **Generate Token. **
7. Copy the token shown on screen, **it will be shown only once. **
8. Save the token in an empty text file at somewhere handy.

# Install `doctl`
`doctl` is a package that allows you to interact with DigitalOcean API via the command line. It is a must to install `doctl` on your droplet to continue the following steps.

1. Update your Arch Linux system[^10][^11]
```bash
sudo pacman -Syu
```
- `sudo` runs the command as an administrator.
- `pacman` is the package manager for Arch Linux.
- `S` synchronizes the package.
- `y` refreshes the package databases.
- `u` upgrades all installed packages to latest versions.

2. Download `doctl` package
```bash
sudo pacman -Sy doctl
```

# Use the API token to grant account access to `doctl`[^4][^12]
1. Use the API token to grant `doctl` access to your DigitalOcean account. 
```bash
   doctl auth init
```
- `doctl auth init` Initialize `doctl` to use a specific account
- After the command you will be asked to type in your DigitalOcean API token.


4. Pass in the token string you saved from Digitalocean when prompted by `doctl auth init`
   
5. Restart your terminal.
   
6. Validate that `doctl` is working, the command line will show the information of your account you linked.
   ```bash
   doctl account get
   ```
- `doctl account get` Retrieve account profile details


> **Note:** Output should see a table like below
> ![](./images/Pasted%20image%2020240923232532.png) 



5. To confirm the connection, can create a test droplet as follow
   ```bash
   doctl compute droplet create --region sfo3 --image aapanel --size s-1vcpu-1gb droplet-test
	```
- It is just for testing, we will go through the detail in the later steps

> **Note:** Output should see a table like below
> ![](./images/Pasted%20image%2020240926202709.png)


6. Check if the droplet created successfully and copy the ID of the test droplet
   ```bash
   doctl compute droplet list
	```


7. Delete the test droplet
      ```bash
   doctl compute droplet delete <droplet ID>
	```

# Create SSH
Why do we need SSH key?[^5]
`ssh-keygen` is a tool for creating new authentication key pairs for SSH. It is used for automating logins, single sign-on, and for authenticating hosts.
The key pair include two keys: 
- Authorized keys are public keys that grant access. They are like locks that need to have the right pairing private key to open and build the connection between two devices.
- Identity keys are private keys that an SSH client uses to authenticate itself when logging into an SSH server. They are like keys that can open many locks, which means the device who has private keys can connect to many different devices which have the matching public keys.

Authorized keys and identity keys are jointly called user keys. They relate to user authentication, as opposed to host keys that are used for host authentication.[^5]

1. Create SSH key[^5]
```bash
ssh-keygen -t ed25519 -f ~/.ssh/do-key -C "your email address"
```
- `ssh-keygen` is a tool for creating new authentication key pairs for SSH.
- `-t ed2219` this is a new algorithm added in OpenSSH. 
- `-f ~/.ssh/your-key-name` is saying the name and the path of your key should be.
- `-C "your email address"` changes the comment of the SSH key, the comment will be included in the key.


1. Get your public key[^13]
```bash
cat ~/.ssh/do-key.pub 
# please change "do-key.pub" to your public key name
# or if you store your ssh key in another folder please change the path
```
- `cat`: concatenate files and print on the standard output


2. Copy the public key printed on screen including your email address

3. Upload the public key to Digitalocean[^6] [^14]
   We need to upload our new public key to Digitalocean so that we can use it later for our connection to the new droplets.
```bash
doctl compute ssh-key import do-key --public-key-file ~/.ssh/do-key.pub
```
- `doctl compute ssh-key import do-key` Import an SSH key from your computer to your account. 
- `--public-key-file` the flag is required to refer the public key file. 
- `~/.ssh/do-key.pub` the path of your public key file. 


4. Check the import succeeded[^6] [^14]
```bash
doctl compute ssh-key list
```
- `doctl compute ssh-key list` List all SSH keys on your account
- Make sure the new ssh-key you just imported is on the list

# Create a config file for cloud-init file
Cloud-init is an industry standard tool that allows you to automate the initialization of your Linux instances. This means that you can use cloud-init to inject a file into your Droplets at deployment that automatically sets up things like new users, firewall rules, app installations, and SSH keys.[^14]

1. Install `neovim`, which is a text editing package. You can use whatever package as long as it can create files and do the text editing [^15]
```bash
sudo pacman -Sy neovim
```


2. Create a cloud-config.yaml file in the folder you want (recommend to store in .ssh folder)[^17]
```bash
nvim your-file-name.yaml # please change the name
```


3. Paste in the following context into your yaml file[^8] [^15] [^7]
```bash
#cloud-config
users:
  - name: user-name # The user's login name, high possible to be arch
    primary_group: group-name #change me
    groups: wheel
    shell: /bin/bash 
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-ed25519 ... #public key here

packages:
  - ripgrep
  - rsync
  - neovim
  - fd
  - less
  - man-db
  - bash-completion
  - tmux
  - doctl

disable_root: true

```
- Press **i** to switch to insert mode
- Press Esc to cancel insert mode
- `:w` save the file
- `:q` close this window
- `:wq` save the file then close the window 
 - **users.name:** should match the username of your Digitalocean user name, likely to be **arch**.
- **users.primary_group:** is like a tag. Define the primary group. Defaults to a new group created name after the user.
- **users.groups:** Optional. Additional groups to add the user to. Defaults to none.
- **packages:** are the modules you want the new droplets to install while they created.
- **sudo:** Defaults to none. Accepts a sudo rule string, a list of sudo rule strings or False to explicitly deny sudo usage.
- **ssh-authorized-keys:**  Optional. Add keys to user's authorized keys file. An error will be raised if no_create_home or system is also set.
- **disable_root:** Set it to `true`. To prevent hackers hack into the server through root user. An option to disable root user access so that only the `example-user` user can access the Droplet.

> **Note:** 
> Please watch out the space of your yaml file, it might cause an error if there's any extra space in the file

# Create a Droplet 
Since we have all the materials ready, we can now create our new droplets by apply all the outputs from former steps.

1. Confirm the image ID
   You've already created an arch Linux droplet, just find the same one on the list and copy the ID.
```bash
doctl compute image list 
```
> **Note:** 
> Find the image we uploaded for arch linux and copy the ID, which is **165086895** in this case
> ![](./images/Pasted%20image%2020240924185154.png)


2. Confirm the ID of your public key[^12]
```bash
doctl compute ssh-key list
```
> **Note:** 
> Find your public key uploaded before and going to use, which is **43494133** in this case
> ![](./images/Pasted%20image%2020240924192047.png)


3. Create a droplet[^7] [^9]
```bash
doctl compute droplet create --image <your image ID> --size s-1vcpu-1gb-amd --region sfo3 --ssh-keys <SSH ID> --user-data-file ~/.ssh/your-file-name.yaml --wait <new-droplet-name1> <new-droplet-name2> <new-droplet-name3>
```
- `doctl compute droplet create` is the command to create droplet.
- `--image` is the tag referring the image you want to use and decide your OS system, should type image ID behind the tag.
- `--size` is the tag  referring the server's size of the CPU, and the CPU option. Can use `doctl compute size list` to find to one you want. We use `s-1vcpu-1gb-amd` which is using basic > 1 AMD CPU for 1 GB.
- `--region` is the server's region and datacenter. We should choose the one that is near to the end users to reduce latency. use `doctl compute region list` to check what datacenters are available. Since we are our own end users, we choose San Francisco, which is the nearest region to us. `sfo1` is not available at the moment, so we can use `sfo2` or `sfo3`.
![](./images/Pasted%20image%2020240926214757.png)


- `--ssh-keys` is the tag referring the SSH key you want to use to connect the droplet, add SSH ID behind.
- `--user-data-file`  is the tag referring the yaml file you want to use to configure your new droplets, add the file path of your yaml file behind.
- `--wait` Tells `doctl` to wait for the Droplets to finish deployment before accepting new commands.
- `new-droplet-name1` add the new droplet names to the end of the command line. If you want to create multiple droplets just add many different droplet names behind.

# Create a config file to automatically build the connection to the new droplet
1. `cd` to .ssh folder
2. Create a config file by using neovim `nvim`[^17]
```bash
nvim config
```


3. Copy and paste the config content[^16][^5]
```bash
Host <name-anything> # change it to the name you want to call

  HostName <000.00.00.000> # change it to the public IP address of your droplet

  User <digitalocean-name> # change it to match with DigitalOcean, possible to be arch 

  PreferredAuthentications publickey

  IdentityFile ~/.ssh/key-name # change the key-name to your ssh private key

  StrictHostKeyChecking no

  UserKnownHostsFile /dev/**null******
```
- `HosName` The actual hostname that should be used to establish the connection. Type in IP address of your new droplet.
- `User` The username to be used for the connection. It should be `arch` which is the same as your Digitalocean username.
- `StrictHostKeyChecking` This option decides whether SSH will automatically add hosts to the `~/.ssh/known_hosts` file. By default, this is set to “ask” meaning that it will warn you if the Host Key received from the remote server does not match the one found in the `known_hosts` file. If you are constantly connecting to a large number of ephemeral hosts (such as testing servers), you may want to turn this to “no”. SSH will then automatically add any hosts to the file.
- - `UserKnownHostsFile`This option specifies the location where SSH will store the information about hosts it has connected to. Usually you do not have to worry about this setting, but you may wish to set this to `/dev/null` so they are discarded if you have turned off strict host checking above.


4. Test the connection 
```bash
ssh name-anything # name-anything is the one you named in the config file
```


# Reference
  
[^3]: DigitalOcean. (n.d.). **Create a personal access token**. DigitalOcean Documentation. [https://docs.digitalocean.com/reference/api/create-personal-access-token/](https://docs.digitalocean.com/reference/api/create-personal-access-token/)
    
[^4]: DigitalOcean. (n.d.). **How to install doctl**. DigitalOcean Documentation. [https://docs.digitalocean.com/reference/doctl/how-to/install/](https://docs.digitalocean.com/reference/doctl/how-to/install/)
    
[^10]: DigitalOcean. (n.d.). _doctl documentation_. GitHub. Retrieved September 25, 2024, from [https://github.com/digitalocean/doctl?tab=readme-ov-file#arch-linux](https://github.com/digitalocean/doctl?tab=readme-ov-file#arch-linux)
    
[^11]:Arch Linux. (n.d.). _Pacman_. ArchWiki. Retrieved September 25, 2024, from [https://wiki.archlinux.org/title/Pacman](https://wiki.archlinux.org/title/Pacman)
    
[^12]:DigitalOcean. (n.d.). _doctl reference_. Retrieved September 25, 2024, from [https://docs.digitalocean.com/reference/doctl/reference/](https://docs.digitalocean.com/reference/doctl/reference/)
    
[^13]:Arch Linux. (n.d.). _cat(1) – Linux man page_. Retrieved September 25, 2024, from [https://man.archlinux.org/man/cat.1.en](https://man.archlinux.org/man/cat.1.en)
    
[^5]: SSH Academy. (n.d.). **SSH keygen**. SSH.com. [https://www.ssh.com/academy/ssh/keygen](https://www.ssh.com/academy/ssh/keygen)
    
[^6]: DigitalOcean. (n.d.). **Import SSH public key**. DigitalOcean Documentation. [https://docs.digitalocean.com/reference/doctl/reference/compute/ssh-key/import/](https://docs.digitalocean.com/reference/doctl/reference/compute/ssh-key/import/)
    
[^14]:DigitalOcean. (n.d.). _doctl compute ssh-key – DigitalOcean_. Retrieved September 25, 2024, from [https://docs.digitalocean.com/reference/doctl/reference/compute/ssh-key/](https://docs.digitalocean.com/reference/doctl/reference/compute/ssh-key/)
    
[^15]:Staubgeborener. (2020, July 9). _digitalocean/api - Reference list for DigitalOcean API_. GitHub. Retrieved September 25, 2024, from [https://gist.github.com/Staubgeborener/02ecfdc38cb30954573595147a96eb1c](https://gist.github.com/Staubgeborener/02ecfdc38cb30954573595147a96eb1c)
    
[^7]: DigitalOcean. (n.d.). **Automate setup with cloud-init**. DigitalOcean Documentation. [https://docs.digitalocean.com/products/droplets/how-to/automate-setup-with-cloud-init/](https://docs.digitalocean.com/products/droplets/how-to/automate-setup-with-cloud-init/)
    
[^8]: Cloud-Init. (n.d.). **Examples: YAML file syntax**. Cloud-Init Documentation. [https://cloudinit.readthedocs.io/en/latest/reference/examples.html](https://cloudinit.readthedocs.io/en/latest/reference/examples.html)
    
[^9]: DigitalOcean. (2020, August 17). **Choosing a data center location**. DigitalOcean Blog. [https://www.digitalocean.com/blog/choosing-a-data-center-location](https://www.digitalocean.com/blog/choosing-a-data-center-location)
    
[^16]:DigitalOcean. (2022, February 22). _How to configure custom connection options for your SSH client_. DigitalOcean Community. Retrieved September 25, 2024, from [https://www.digitalocean.com/community/tutorials/how-to-configure-custom-connection-options-for-your-ssh-client](https://www.digitalocean.com/community/tutorials/how-to-configure-custom-connection-options-for-your-ssh-client)
    
[^17]:Neovim. (n.d.). _Vim index_. Retrieved September 25, 2024, from [https://neovim.io/doc/user/vimindex.html](https://neovim.io/doc/user/vimindex.html)





