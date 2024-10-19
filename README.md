# Filebrowser Setup and Deployment to Ubuntu Server

## Prerequisites

- A server with freshly installed Ubuntu
- SSH access to your Ubuntu Server

## Step 1: Connect

1\. **Connect to Your Ubuntu Server**:

```sh
ssh user@YOUR-UBUNTU-SERVER-IP-ADDRESS
```

## Step 2: Install Docker on Your Ubuntu Server

1\. **Install Docker**:

```sh
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin
```

2\. **Install Docker Compose**:

```sh
sudo apt-get install docker-compose-plugin
sudo curl -L "https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

3\. **Add User to Docker Group**:

```sh
sudo usermod -aG docker $USER
newgrp docker
```

## Step 3: Generate SSH Key and Add to GitHub

1\. **Generate the SSH Key Pair on your Ubuntu Server**:

```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
```

2\. **Add the Public Key to `authorized_keys`**:

```sh
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

3\. **Retrieve the Private Key**:

```sh
cat ~/.ssh/id_ed25519
```

4\. **Copy the content of the private key** to use in GitHub Secrets as SSH_PRIVATE_KEY.

5\. **Add the SSH Public Key to GitHub**:

```sh
cat ~/.ssh/id_ed25519.pub
```

- Go to GitHub and log in to your account.
- In the upper-right corner of any page, click your profile photo, then click **Settings**.
- In the user settings sidebar, click **SSH and GPG keys**.
- Click **New SSH key**.
- In the "Title" field, add a descriptive label for the new key.
- Paste your key into the "Key" field.
- Click **Add SSH key**.
- Confirm your GitHub password if prompted.

## Step 4: Install and Configure Caddy

### Install Caddy

```sh
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install -y caddy
```

### Configure Caddy

```sh
sudo vim /etc/caddy/Caddyfile
```

Add the following configuration:

```caddyfile
silverbullet.donce.dev {
        tls adom.donatas@gmail.com
        request_body {
                max_size 100MB
        }
        reverse_proxy 127.0.0.1:3303
}
```

**Restart Caddy:**

```sh
sudo systemctl restart caddy
```

Check the Caddy service status:

```sh
sudo systemctl status caddy
```

## Step 5: After Server Setup You Can Deploy Repo via Github

1\. **Add Secrets to GitHub**:

- Go to your GitHub repository.
- Navigate to `Settings` > `Secrets and variables` > `Actions`.
- Add the following secrets:
- `SSH_PRIVATE_KEY`: Your private SSH key.
- `UBUNTU_SERVER_IP`: Your Ubuntu Server IP address.
- `USERNAME`: Your Silverbullet username.
- `PASSWORD`: Your Silverbullet password.

These secrets are used in the GitHub Actions workflow to securely deploy your application.

2\. **Now You Can Commit Changes to the Github Repository and Deploy it should automatically deploy to Ubuntu Server**

3\. **After Deployment You Can Check the Service Status with this Command Inside of Ubuntu Server**

```sh
docker service ls
```
