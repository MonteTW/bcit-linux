
# Create an API Token
[reference](https://docs.digitalocean.com/reference/api/create-personal-access-token/)

1. login to digitalocean control panel
2. click API in the left menu
3. Click **Generate New Token** in the Personal access tokens section
4. Type in the name of the token and select 90 days expiration
5. Choose **Full Access** for the Scopes
6. Click **Generate Token**
7. Copy the token shown on screen, it will be shown only once
8. Save the token in a empty text file in .ssh directory 

# Install ```doctl```
[reference](https://docs.digitalocean.com/reference/doctl/how-to/install/)

1. Install ```wget```
   ```bash
sudo pacman -Sy wget
```

2. Check if ```wget``` is installed and print more information about the tool run
```bash
pacman -Qi wget
```

3. Download the ```doctl``` by using code below:
   ```bash
cd ~
wget https://github.com/digitalocean/doctl/releases/download/v1.110.0/doctl-1.110.0-linux-amd64.tar.gz
	```
4. Extract the binary:
   ```bash
tar xf ~/doctl-1.110.0-linux-amd64.tar.gz
	```
5. Move the ```doctl``` binary into a dedicated directory and add it to your system’s path by running:
   ```bash
sudo mv ~/doctl /usr/local/bin
	```

# Use the API token to grant account access to ```doctl```
1. Use the API token to grant ```doctl``` access to your DigitalOcean account, give this authentication context a name. 
   ```bash
   doctl auth init --context <NAME>
   ```
2. Pass in the token string you saved from Digitalocean when prompted by ```doctl auth init``` 
3. Restart your terminal
4. Validate that ```doctl``` is working
   ```bash
   doctl account get
   ```

>[!note] 
>should see a table like below
![[Pasted image 20240923232532.png]] 

5. To confirm the connection, can create a test droplet as follow
   ```bash
   doctl compute droplet create --region sfo3 --image aapanel --size s-1vcpu-1gb droplet-test
	```

6. Check if the droplet created successfully
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

2. get your public key
```bash
cat ~/.ssh/do-key.pub
```

copy the public key print on screen

3. Upload the public key to Digitalocean 
```bash
doctl compute ssh-key import do-key --public-key-file ~/.ssh/do-key.pub
```
4. Check the import succeeded
```bash
doctl compute ssh-key list
```

# Create a config file for cloud-init file
1. Install ```neovim```
```bash
sudo pacman -S neovim
```

2. Create a cloud-config.yaml file
```bash
nvim cloud-config.yaml
```

paste in the following context into the yaml file
```bash
#cloud-config
users:
  - name: user-name #change me
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
>[!note]Commands
>**name** should match the username of your Digitalocean user name, likely to be **arch**
>**group-name** is like a tag to tag the droplets who share the same traits or for the same project
>**public key** should paste the one that match with the private key of the droplet you're using
>**packages** are the modules you want the new droplets to install while they created 

# Create a Droplet 
[reference](https://docs.digitalocean.com/products/droplets/how-to/create/)

[How to select the region](https://www.digitalocean.com/blog/choosing-a-data-center-location)

1. Confirm the ID of your image ID
```bash
doctl compute image list 
```
>[!note] Look for the one you uploaded
>Find the one we uploaded for arch linux and copy the ID, which is 165086895 in this case
![[Pasted image 20240924185154.png]]
2. Confirm the ID of your public key
```bash
doctl compute ssh-key list
```
>[!note] Find your public key
>Find the one we uploaded before and going to use, which is 43494133 in this case
![[Pasted image 20240924192047.png]]

4. Create a droplet
```bash
doctl compute droplet create --image 165086895 --size s-1vcpu-1gb-amd --region sfo3 --ssh-keys 43494133 --user-data-file ~/.ssh/cloud-config.yaml --wait as1-test

```


# Create a config file to automate build the connection to the new droplet
[reference](https://docs.digitalocean.com/products/droplets/how-to/automate-setup-with-cloud-init/)

1. ```cd``` to .ssh folder
2. Create a config file
```bash
nvim config
```
3. Copy and paste the config content
```
Host art1-test1

  HostName 164.92.71.207

  User arch

  PreferredAuthentications publickey

  IdentityFile ~/.ssh/ar1-key

  StrictHostKeyChecking no

  UserKnownHostsFile /dev/null
```

4. Test the connection
```bash
ssh ast1-test1
```


# Source
