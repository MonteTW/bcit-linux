
# Create an API Token
1. login to digitalocean control panel.
2. click API in the left menu.
3. Click **Generate New Token** in the Personal access tokens section.
4. Type in the name of the token and select 90 days expiration.
5. Choose **Full Access** for the Scopes.
6. Click **Generate Token.**
7. Copy the token shown on screen, it will be shown only once.
8. Save the token in an empty text file at somewhere handy.

# Install `doctl`
`doctl` allows you to interact with the DigitalOcean API via the command line.

1. update your Arch Linux system
```bash
sudo pacman -Syu
```
- `sudo` runs the command with administrative privileges.
- `pacman` is the package manager for Arch Linux.
- `-Syu` performs a full system upgrade:
    - `S` synchronizes the package databases.
    - `y` refreshes the package databases.
    - `u` upgrades all installed packages to their latest versions.

2. Download `doctl` package
```bash
sudo pacman -Sy doctl
```

# Use the API token to grant account access to `doctl`
1. Use the API token to grant `doctl` access to your DigitalOcean account. 
```bash
   doctl auth init
```
`doctl auth init`: After the command you will be asked to type in your DigitalOcean API token.
4. Pass in the token string you saved from Digitalocean when prompted by `doctl auth init`
   
5. Restart your terminal.
   
6. Validate that `doctl` is working, the command line will show the information of your account you linked.
   ```bash
   doctl account get
   ```
- `doctl` allows you to interact with DigitalOcean resources like, Droplets, from your terminal.
- `account`: This is referring to the DigitalOcean account you are authenticated with. It deals with account-level information, such as your email, API token usage, droplet limits, and account status.
- `get`: This is the action or sub-command. It fetches or retrieves the information associated with the `account` resource, displaying key details about your account.


> [!NOTE] Output 
> should see a table like below
> ![](./images/Pasted%20image%2020240923232532.png) 

5. To confirm the connection, can create a test droplet as follow
   ```bash
   doctl compute droplet create --region sfo3 --image aapanel --size s-1vcpu-1gb droplet-test
	```
- it is just for testing, we will go through the detail in the later steps

> [!NOTE] Output 
> should see a table like below
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
1. create SSH key
```bash
ssh-keygen -t ed25519 -f ~/.ssh/do-key -C "your email address"
```
- `ssh-keygen` is a tool for creating new authentication key pairs for SSH.
- `-t ed2219` is saying the type of key to be created. `ed25519` is a modern, highly secure public-key cryptographic system, which is considered more secure than older key types like RSA.
- `-f ~/.ssh/your-key-name` is saying the name and the path of your key should be.
- `-C "your email address"` changes the comment of the SSH key, the comment will be included in the key.

1. get your public key
```bash
cat ~/.ssh/do-key.pub 
# please change "do-key.pub" to your public key name
# or if you store your ssh key in another folder please change the path
```

copy the public key printed on screen

3. Upload the public key to Digitalocean 
```bash
doctl compute ssh-key import do-key --public-key-file ~/.ssh/do-key.pub
```
- `doctl compute ssh-key import do-key` Import an SSH key from your computer to your account.
- `--public-key-file` the flag is required to refer the public key file.
- `~/.ssh/do-key.pub` the path of your public key file.

4. Check the import succeeded
```bash
doctl compute ssh-key list
```

# Create a config file for cloud-init file
1. Install `neovim`, which is a package for creating files and editing the content of files
```bash
sudo pacman -S neovim
```

2. Create a cloud-config.yaml file in the folder you want (recommend to store in .ssh folder)
```bash
nvim your-file-name.yaml # please change the name
```

3. paste in the following context into the yaml file
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
- **disable_root:** Set it to `true`. To prevent hackers hack into the server through root user.

> [!NOTE]
> Please watch out the space of your yaml file, it might cause an error if there's any extra space in the file
# Create a Droplet 

1. Confirm the ID of your image ID
```bash
doctl compute image list 
```
> [!NOTE] Look for the one you uploaded
> Find the one we uploaded for arch linux and copy the ID, which is **165086895** in this case
> ![](./images/Pasted%20image%2020240924185154.png)
2. Confirm the ID of your public key
```bash
doctl compute ssh-key list
```
> [!NOTE] Find your public key
> Find the one we uploaded before and going to use, which is **43494133** in this case
> ![](./images/Pasted%20image%2020240924192047.png)

3. Create a droplet
```bash
doctl compute droplet create --image your-image-id --size s-1vcpu-1gb-amd --region sfo3 --ssh-keys 43494133 --user-data-file ~/.ssh/your-file-name.yaml --wait new-droplet-name1 new-droplet-name2 new-droplet-name3
```
- `doctl compute droplet create` is the command to create droplet.
- `--image` is the tag referring the image you want to use and decide your OS system, should type image ID behind the tag.
- `--size` is the tag  referring the server's size of the CPU, and the CPU option. Can use `doctl compute size list` to find to one you want. We use `s-1vcpu-1gb-amd` which is using basic > 1 AMD CPU for 1 GB.
- `--region` is the server's region and datacenter. We should choose the one that is near to the end users to reduce latency. use `doctl compute region list` to check what datacenters are available. Since we are our own end users, we choose San Francisco, which is the nearest region to us. `sfo1` is not available at the moment, so we can use `sfo2` or `sfo3`.
![](./images/Pasted%20image%2020240926214757.png)


- `--ssh-keys` is the tag referring the SSH key you want to use to connect the droplet, add SSH ID behind.
- `--user-data-file`  is the tag referring the yaml file you want to use to configure your new droplets, add the file path of your yaml file behind.
- `--wait` Tells `doctl` to wait for the Droplets to finish deployment before accepting new commands. Add 
- `new-droplet-name1` add the new droplet names to the end of the command line. If you want to create multiple droplets just add many different droplet names behind.

# Create a config file to automate build the connection to the new droplet
1. `cd` to .ssh folder
2. Create a config file by using neovim `nvim`
```bash
nvim config
```

3. Copy and paste the config content
```bash
Host name-anything # change it to the name you want to call

  HostName 000.00.00.000 # change it to the public IP address of your droplet

  User name-after-digitalocean # change it to match with DigitalOcean, possible to be arch 

  PreferredAuthentications publickey

  IdentityFile ~/.ssh/key-name # change the key-name to your ssh private key

  StrictHostKeyChecking no

  UserKnownHostsFile /dev/**null******
```

4. Test the connection
```bash
ssh name-anything # name-anything is the one you named in the config file
```


# Reference

- DigitalOcean. (n.d.). **doctl**. DigitalOcean Documentation. [https://docs.digitalocean.com/reference/doctl/](https://docs.digitalocean.com/reference/doctl/)
    
- DigitalOcean. (n.d.). **Installing doctl**. GitHub. [https://github.com/digitalocean/doctl?tab=readme-ov-file#installing-doctl](https://github.com/digitalocean/doctl?tab=readme-ov-file#installing-doctl)
    
- DigitalOcean. (n.d.). **Create a personal access token**. DigitalOcean Documentation. [https://docs.digitalocean.com/reference/api/create-personal-access-token/](https://docs.digitalocean.com/reference/api/create-personal-access-token/)
    
- DigitalOcean. (n.d.). **How to install doctl**. DigitalOcean Documentation. [https://docs.digitalocean.com/reference/doctl/how-to/install/](https://docs.digitalocean.com/reference/doctl/how-to/install/)
    
- SSH Academy. (n.d.). **SSH keygen**. SSH.com. [https://www.ssh.com/academy/ssh/keygen](https://www.ssh.com/academy/ssh/keygen)
    
- DigitalOcean. (n.d.). **Import SSH public key**. DigitalOcean Documentation. [https://docs.digitalocean.com/reference/doctl/reference/compute/ssh-key/import/](https://docs.digitalocean.com/reference/doctl/reference/compute/ssh-key/import/)
    
- DigitalOcean. (n.d.). **Automate setup with cloud-init**. DigitalOcean Documentation. [https://docs.digitalocean.com/products/droplets/how-to/automate-setup-with-cloud-init/](https://docs.digitalocean.com/products/droplets/how-to/automate-setup-with-cloud-init/)
    
- Cloud-Init. (n.d.). **Examples: YAML file syntax**. Cloud-Init Documentation. [https://cloudinit.readthedocs.io/en/latest/reference/examples.html](https://cloudinit.readthedocs.io/en/latest/reference/examples.html)
    
- DigitalOcean. (n.d.). **Automate setup with cloud-init**. DigitalOcean Documentation. [https://docs.digitalocean.com/products/droplets/how-to/automate-setup-with-cloud-init/](https://docs.digitalocean.com/products/droplets/how-to/automate-setup-with-cloud-init/)
    
- DigitalOcean. (2020, August 17). **Choosing a data center location**. DigitalOcean Blog. [https://www.digitalocean.com/blog/choosing-a-data-center-location](https://www.digitalocean.com/blog/choosing-a-data-center-location)