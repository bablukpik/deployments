# How to Deploy NodeJS Apps to VPS Server

To deploy your NodeJS apps to VPS server, you can use your server IP, you can create a root password and enter the server with your IP address and password credentials. But the more secure way is using an SSH key.

## Creating SSH Key

### For MAC OS / Linux / Windows 10 (with openssh)

1. Launch the Terminal app.
2. `ssh-keygen -t rsa`
3. Press `ENTER` to store the key in the default folder /Users/bablu/.ssh/id_rsa or ~/.ssh/id_rsa).

4. Type a passphrase (characters will not appear in the terminal).

5. Confirm your passphrase to finish SSH Keygen. You should get an output that looks something like this:

```Your identification has been saved in /Users/bablu/.ssh/id_rsa or ~/.ssh/id_rsa.
Your public key has been saved in /Users/bablu/.ssh/id_rsa.pub or ~/.ssh/id_rsa.pub.
The key fingerprint is:
ae:89:72:0b:85:da:5a:f4:7c:1f:c2:43:fd:c6:44:30 bablu@mac.local
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|         .       |
|        E .      |
|   .   . o       |
|  o . . S .      |
| + + o . +       |
|. + o = o +      |
| o...o * o       |
|.  oo.o .        |
+-----------------+
```

6. Copy your public SSH Key to your clipboard using the following code:
   `pbcopy < ~/.ssh/id_rsa.pub`

### For Windows

1. Download PuTTY and PuTTYgen.
2. Open up PuTTYgen and click the `Generate`.
3. Copy your key.
4. Enter a key passphrase and confirm.
5. Save the private key.

## Connection

After copying the SSH Key go the to hosting service provider dashboard and paste your key and save. After,

### For MAC OS / Linux

```bash
ssh root@<server ip address>
```

### For Windows

1. Open the PuTTY app.
2. Enter your IP address.
3. Open the following section:
   Connection - SSH - Auth
4. Browse the folders and choose your private key.

## First Configuration

### Deleting apache server

If you have an apache server installed, you can delete it by following the steps below:

```
systemctl stop apache2
```

```bash
sudo systemctl disable apache2
```

```bash
sudo apt remove apache2
```

to delete related dependencies:

```bash
sudo apt autoremove
```

### Cleaning and updating server

```bash
sudo apt clean all && sudo apt update && sudo apt dist-upgrade
```

```bash
sudo rm -rf /var/www/html
```

### Installing Nginx

```bash
sudo apt install nginx
```

### Installing and configure Firewall

```bash
sudo apt install ufw
```

```bash
sudo ufw allow 'Nginx Full' && sudo ufw allow ssh
```

```bash
sudo ufw enable
```

## First Page

#### Delete the default server configuration

```bash
sudo rm /etc/nginx/sites-available/default
```

```bash
sudo rm /etc/nginx/sites-enabled/default
```

#### First configuration

```bash
sudo vim /etc/nginx/sites-available/w3public
```

```bash
server {
  listen 80;

  location / {
        root /var/www/w3public;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }
}

```

```bash
sudo ln -s /etc/nginx/sites-available/w3public /etc/nginx/sites-enabled/w3public
```

```bash
sudo systemctl restart nginx
```

##### Write your fist message

```bash
sudo vim /var/www/w3public/index.html
```

##### Start Nginx and check the page

```bash
sudo systemctl start nginx
```

## Uploading Projects

### Using Git

```bash
sudo apt install git
```

```
sudo mkdir w3public
```

```bash
cd w3public
```

```
git clone <your repository>
```

### Using scp

Copy all files and folders from the **dist** folder to VPS:

```bash
scp -r dist/* ubuntu@35.185.247.77:/var/www/w3public/frontend
```

Copy a Specific File to VPS:

```bash
scp dist/index.html ubuntu@35.185.247.77:/var/www/w3public/frontend
```

Or if your index.html file is in the root folder:

```bash
scp index.html ubuntu@35.185.247.77:/var/www/w3public/frontend
```

## Nginx Configuration for new Apps

```bash
sudo vim /etc/nginx/sites-available/w3public
```

```bash
location /api {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
  }
```

##### If you check the location /api you are going to get "502" error which is good. Our configuration works. The only thing we need to is running our app

```bash
apt install nodejs
```

```bash
cd api
```

```bash
npm install
```

```bash
nano .env
```

##### Copy and paste your env file

```bash
node index.js
```

#### But if you close your ssh session here. It's gonna kill this process. To prevent this we are going to need a package which is called `pm2`

```bash
npm i -g pm2
```

```bash
pm2 startup
```

```bash
pm2 save
```

Let's create a new pm2 instance

```bash
pm2 start --name api-prod index.js
```

```bash
pm2 status
```

You should see a list of running processes like so:

```markdown
| id  | name     | namespace | version | mode | pid    | uptime | 𝛑   | status | cpu | mem     | user   | watching |
| --- | -------- | --------- | ------- | ---- | ------ | ------ | --- | ------ | --- | ------- | ------ | -------- |
| 0   | api-prod | default   | 1.0.0   | fork | 153771 | 65s    | ⚙️  | online | 0%  | 100.5mb | ubuntu | disabled |
```

If you want to restart the process or API you created, you can use the following command:

```bash
pm2 restart api-prod
```

## React App Deployment

```bash
cd ../client
```

```bash
nano .env
```

Paste your env file.

```bash
npm i
```

Let's create the build file

```bash
npm run build
```

Right now, we should move this build file into the main web file

```bash
rm -rf /var/www/w3public/*
```

```bash
mkdir /var/www/w3public/client
```

```bash
cp -r build/* /var/www/w3public/client
```

Let's make some server configuration

```bash
 location / {
    root /var/www/w3public/client/;
    index  index.html index.htm;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    try_files $uri $uri/ /index.html;
  }
```

### Enable Permission

```bash
sudo chown -R $USER:www-data /var/www/
```

```bash
sudo chmod -R 777 /var/www/
```

### Adding Domain

1 - Make sure that you created your A records on your domain provider website.

2 - Change your pathname from Router

3 - Change your env files and add the new API address

4 - Add the following server config

```bash
server {
  listen 80;
  server_name w3public.com; # If you have www.w3public.com, you can use `server_name w3public.com www.w3public.com;`

  location / {
    root /var/www/w3public/client;
    index  index.html index.htm;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    try_files $uri $uri/ /index.html;
  }
}

server {
  listen 80;
  server_name api.w3public.com;
  location / {
    proxy_pass http://localhost:8000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}

server {
  listen 80;
  server_name admin.w3public.com;
  location / {
    root /var/www/w3public/admin;
    index  index.html index.htm;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    try_files $uri $uri/ /index.html;
  }
}
```

### VHOST Examples

There are two ways to create a vhost server block.

#### One Server Block for Different Locations (One way)

````bash
server {
  listen 80;
  server_name opencv.w3public.com;

  location / {
    root /var/www/dsd-sji/frontend;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;

    # WebSocket proxy configurations
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }

  location /api {
    proxy_pass http://localhost:8000;

    # WebSocket proxy configurations
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;

    # Timeout settings
    proxy_connect_timeout   20s;
    proxy_send_timeout      20s;
    proxy_read_timeout      20s;
    send_timeout            20s;
  }
}

#### Different server blocks for each location (Another way)

```bash
server {
  listen 80;
  server_name example.com www.example.com;

  location / {
    root /var/www/w3public/client;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;

    # WebSocket proxy configurations
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}

server {
  listen 80;
  server_name api.example.com;
  location / {
    proxy_pass http://198.12.124.82:3000;

    # WebSocket proxy configurations
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}

server {
  listen 80;
  server_name admin.example.com;
  location / {
    root /var/www/w3public/admin;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;

    # WebSocket proxy configurations
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}
````

**Note**: If you don't have a domain, you can use the IP address of your VPS server, in this case, you have to use a single server block for all the locations:

```bash
server {
  listen 80;

  location / {
    root /var/www/w3public/frontend;
    index  index.html index.htm;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    try_files $uri $uri/ /index.html;
  }

  location /api {
    proxy_pass http://localhost:8000;

    # Set the maximum allowed size of the client request body to 100MB
    client_max_body_size 100M;

    # WebSocket proxy configurations
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;

    # Timeout settings
    proxy_connect_timeout   120s;
    proxy_send_timeout      120s;
    proxy_read_timeout      120s;
    send_timeout            120s;
  }
}
```

## SSL Certification

To get SSL Certificate using Certbot, [here is the step by step guide to get SSL certification for your domain](https://certbot.eff.org/instructions).

I'm providing the steps below for reference using snap:

1. **SSH into the server**
   SSH into the server running your HTTP website as a user with sudo privileges.

2. **Remove certbot-auto and any Certbot OS packages**
   If you have any Certbot packages installed using an OS package manager like apt, dnf, or yum, you should remove them before installing the Certbot snap to ensure that when you run the command certbot the snap is used rather than the installation from your OS package manager. The exact command to do this depends on your OS, but common examples are `sudo apt-get remove certbot`, `sudo dnf remove certbot`, or `sudo yum remove certbot`.

3. **Install Certbot**
   Run this command on the command line on the machine to install Certbot.

```bash
sudo snap install --classic certbot
```

4. **Prepare the Certbot command**
   Execute the following instruction on the command line on the machine to ensure that the certbot command can be run.

```bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

5. **Choose how you'd like to run Certbot**
   Either get and install your certificates...
   Run this command to get a certificate and have Certbot edit your nginx configuration automatically to serve it, turning on HTTPS access in a single step.

```bash
sudo certbot --nginx
```

Or, just get a certificate
If you're feeling more conservative and would like to make the changes to your nginx configuration by hand, run this command.

```bash
sudo certbot certonly --nginx
```

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

6. **Test automatic renewal**
   The Certbot packages on your system come with a cron job or systemd timer that will renew your certificates automatically before they expire. You will not need to run Certbot again, unless you change your configuration. You can test automatic renewal for your certificates by running this command:

```bash
sudo certbot renew --dry-run
```

The command to renew certbot is installed in one of the following locations:

- /etc/crontab/
- /etc/cron._/_
- systemctl list-timers

7. **Confirm that Certbot worked**
   To confirm that your site is set up properly, visit https://example.com/ in your browser and look for the lock icon in the URL bar.
