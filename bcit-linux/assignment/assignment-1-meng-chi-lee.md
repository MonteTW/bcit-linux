
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

# Install doctl
[reference](https://docs.digitalocean.com/reference/doctl/how-to/install/)

1. Install wget
   ```powershell
sudo pacman -Sy wget
```

2. Check if wget is installed and print more information about the tool run
```powershell
pacman -Qi wget
```

3. Download the doctl by using code below:
   ```powershell
cd ~
wget https://github.com/digitalocean/doctl/releases/download/v1.110.0/doctl-1.110.0-linux-amd64.tar.gz
	```
4. Extract the binary:
   ```powershell
tar xf ~/doctl-1.110.0-linux-amd64.tar.gz
	```
5. Move the doctl binary into a dedicated directory and add it to your system’s path by running:
   ```powershell
sudo mv ~/doctl /usr/local/bin
	```

# Use the API token to grant account access to doctl
1. Use the API token to grant doctl access to your DigitalOcean account, give this authentication context a name. 
   ```powershell
   doctl auth init --context <NAME>
   ```
2. Pass in the token string you saved from Digitalocean when prompted by doctl auth init
3. Restart your terminal
4. Validate that doctl is working
   ```powershell
   doctl account get
   ```

note: should see a table like below
![[Pasted image 20240923232532.png]] 

5. To confirm the connection, can create a test droplet as follow
   ```powershell
   doctl compute droplet create --region sfo3 --image aapanel --size s-1vcpu-1gb droplet-test
	```

6. Check if the droplet created successfully
   ```powershell
   doctl compute droplet list
	```

7. Delete the test droplet
      ```powershell
   doctl compute droplet delete <droplet ID>
	```

# Create SSH
1. create SSH key
```powershell
ssh-keygen -t ed25519 -f ~/.ssh/do-key -C "your email address"
```

2. get your public key
```powershell
cat ~/.ssh/do-key.pub
```

copy the public key print on screen

3. Upload the public key to Digitalocean 
```powershell
doctl compute ssh-key import do-key --public-key-file ~/.ssh/do-key.pub
```
4. Check the import succeeded
```powershell
doctl compute ssh-key list
```

5. Install neovim
```powershell
sudo pacman -S neovim
```

6. Create a cloud-config.yaml file

# Create a Droplet Using the CLI
[reference](https://docs.digitalocean.com/products/droplets/how-to/create/)

[How to select the region](https://www.digitalocean.com/blog/choosing-a-data-center-location)


# Automate Droplet with Cloud-init
[reference](https://docs.digitalocean.com/products/droplets/how-to/automate-setup-with-cloud-init/)


