# How to Deploy MERN Stack Apps to VPS Server

## Table of Contents

1. [Creating SSH Key](#creating-ssh-key)
2. [Connection](#connection)
3. [First Configuration](#first-configuration)
4. [Uploading Projects](#uploading-projects)
5. [Nginx Configuration](#nginx-configuration-for-new-apps)
6. [Installing Node.js](#installing-nodejs)
7. [React App Deployment](#react-app-deployment)
8. [Adding Domain](#adding-domain)
9. [Installing MongoDB](#installing-mongodb)
10. [Managing Environment Variables](#managing-environment-variables)
11. [Additional Security Measures](#additional-security-measures)
12. [Backup Strategy](#backup-strategy)
13. [Continuous Deployment](#continuous-deployment)
14. [Monitoring and Logging](#monitoring-and-logging)
15. [Scaling Your Application](#scaling-your-application)
16. [Database Optimization](#database-optimization)
17. [Containerization with Docker](#containerization-with-docker)
18. [Troubleshooting Common Issues](#troubleshooting-common-issues)
19. [Setting Up HTTPS with Let's Encrypt](#setting-up-https-with-lets-encrypt)
20. [MERN Stack Security Best Practices](#mern-stack-security-best-practices)
21. [Performance Optimization](#performance-optimization)

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

### Delete the default server configuration

```bash
sudo rm /etc/nginx/sites-available/default
```

```bash
sudo rm /etc/nginx/sites-enabled/default
```

### First configuration

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

### Write your fist message

```bash
sudo vim /var/www/w3public/index.html
```

### Start Nginx and check the page

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

### If you check the location /api you are going to get "502" error which is good. Our configuration works. The only thing we need to is running our app

**Installing Node.js**

```bash
sudo apt install nodejs
```

Or

It's recommended to use a version manager like nvm to install Node.js. This allows you to easily switch between Node.js versions if needed.

Here is the step by step guide link to install NVM.

[Install NVM](https://github.com/nvm-sh/nvm#installation-and-update)

**I'm providing the steps below for reference**:

Run the command below that downloads a script and runs it. The script clones the nvm repository to ~/.nvm, and attempts to add the NVM global variables to the correct profile file (~/.bash_profile, ~/.zshrc, ~/.profile, or ~/.bashrc).

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

Check if NVM is installed

```bash
nvm --version
```

If it doesn't show the version number, you can add the following line to the end of your ~/.bashrc file:

```bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

And then run the following command to reload the bash profile:

```bash
source ~/.bashrc
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

### Copy and paste your env file

```bash
node index.js
```

### But if you close your ssh session here. It's gonna kill this process. To prevent this we are going to need a package which is called `pm2`

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

## Setting Up HTTPS with Let's Encrypt

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

## Installing MongoDB

MongoDB is a crucial part of the MERN stack. Here's how to install it:

### Reload local package database

```bash
sudo apt-get update
```

### Install the MongoDB packages

```bash
sudo apt-get install -y mongodb-org
```

### Start MongoDB

```bash
sudo systemctl start mongod
```

### Enable MongoDB to start on boot

```bash
sudo systemctl enable mongod
```

## Managing Environment Variables

It's important to manage environment variables securely in production:

1. Create a `.env` file in your project root (if not already present).
2. Add all your environment variables to this file.
3. Install the `dotenv` package: `npm install dotenv`
4. In your main application file (e.g., `app.js` or `server.js`), add:

```javascript
require("dotenv").config();
```

5. Use environment variables in your code like this: `process.env.VARIABLE_NAME`

Never commit your `.env` file to version control. Instead, provide a `.env.example` file with dummy values.

## Additional Security Measures

1. Disable root SSH access:
   Edit `/etc/ssh/sshd_config` and set `PermitRootLogin no`

2. Use strong passwords and consider using SSH key authentication only.

3. Keep your system updated:

   ```bash
   sudo apt update && sudo apt upgrade
   ```

4. Install and configure fail2ban to protect against brute-force attacks:
   ```bash
   sudo apt install fail2ban
   sudo systemctl enable fail2ban
   sudo systemctl start fail2ban
   ```

## Backup Strategy

1. Database Backup:
   Set up a cron job to regularly backup your MongoDB database:

   ```bash
   0 2 * * * mongodump --out /path/to/backup/directory
   ```

2. Application Files Backup:
   Use rsync to backup your application files:

   ```bash
   rsync -avz /path/to/your/app /path/to/backup/directory
   ```

3. Consider using a cloud storage service like AWS S3 or Google Cloud Storage for off-site backups.

These additions should provide a more comprehensive guide for deploying and maintaining a MERN stack application on a VPS.

## Continuous Deployment

Continuous Deployment (CD) automates the process of deploying your application to your VPS whenever changes are pushed to your main branch. Here's how to set it up using GitHub Actions:

1. **Create a GitHub Actions workflow file**:
   In your repository, create a file at `.github/workflows/deploy.yml` with the following content:

   ```yaml
   name: Deploy to VPS
   
   on:
     push:
       branches: [ main ]
   
   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
       - uses: actions/checkout@v2
       
       - name: Install Node.js
         uses: actions/setup-node@v2
         with:
           node-version: '14'
           
       - name: Install dependencies
         run: npm ci
         
       - name: Build application
         run: npm run build
         
       - name: Deploy to VPS
         uses: appleboy/ssh-action@master
         with:
           host: ${{ secrets.HOST }}
           username: ${{ secrets.USERNAME }}
           key: ${{ secrets.SSH_PRIVATE_KEY }}
           script: |
             cd /path/to/your/app
             git pull origin main
             npm ci
             npm run build
             pm2 restart all
   ```

2. **Set up GitHub Secrets**:
   In your GitHub repository, go to Settings > Secrets and add the following secrets:
   - `HOST`: Your VPS IP address
   - `USERNAME`: Your VPS username
   - `SSH_PRIVATE_KEY`: Your SSH private key

3. **Configure your VPS**:
   Ensure your VPS is set up to accept deployments:
   - The repository is cloned on the VPS
   - PM2 is installed and configured
   - Proper permissions are set for the deployment user

4. **Test the workflow**:
   Push a change to your main branch and check the Actions tab in your GitHub repository to see the deployment in action.

Remember to adjust the deployment script according to your specific setup and requirements. This setup assumes you're using PM2 to manage your Node.js processes.

For more complex setups, consider using deployment tools like Ansible, Capistrano, or Docker with container orchestration platforms.

## Monitoring and Logging

Proper monitoring and logging are crucial for maintaining your deployed application:

1. **Application Logging**

   - Use a logging library like Winston or Morgan
   - Configure PM2 to manage logs: `pm2 install pm2-logrotate`

2. **Server Monitoring**

   - Install and configure Prometheus for metrics collection
   - Set up Grafana for visualization

3. **Uptime Monitoring**
   - Use services like UptimeRobot or Pingdom to monitor your application's availability

## Scaling Your Application

As your application grows, you might need to scale:

1. **Vertical Scaling**

   - Upgrade your VPS to a more powerful instance

2. **Horizontal Scaling**

   - Set up a load balancer (e.g., HAProxy or Nginx)
   - Deploy multiple instances of your Node.js application
   - Use a MongoDB replica set for database redundancy

3. **Caching**
   - Implement Redis for caching frequently accessed data

## Database Optimization

Optimize your MongoDB for production:

1. **Indexing**

   - Create appropriate indexes for frequently queried fields

2. **Data Modeling**

   - Design your schema for efficient querying and updates

3. **Monitoring**

   - Use MongoDB Compass or MongoDB Atlas for database monitoring

4. **Regular Maintenance**
   - Schedule regular backups
   - Perform periodic data cleanup and archiving

## Containerization with Docker

Consider using Docker for easier deployment and scaling:

1. **Dockerize Your Application**

   - Create Dockerfiles for your Node.js app and React frontend
   - Use docker-compose for orchestrating multiple containers

2. **Container Management**

   - Consider using Kubernetes for advanced container orchestration

3. **CI/CD with Docker**
   - Integrate Docker builds into your CI/CD pipeline

Remember to keep your deployment guide updated as you make changes to your infrastructure or deployment process.

## MERN Stack Security Best Practices

1. **Input Validation**: Use libraries like Joi or express-validator to validate user inputs.

2. **Authentication**: Implement JWT (JSON Web Tokens) for stateless authentication.

3. **Authorization**: Use middleware to check user roles and permissions.

4. **CSRF Protection**: Implement CSRF tokens in your forms.

5. **XSS Prevention**: Use libraries like DOMPurify to sanitize user-generated content.

6. **CORS Configuration**: Set up proper CORS policies to restrict access to your API.

7. **Rate Limiting**: Implement rate limiting to prevent abuse of your API.

8. **Secure Headers**: Use Helmet.js to set secure HTTP headers.

## Performance Optimization

1. **Server-Side Rendering (SSR)**: Consider using Next.js for SSR to improve initial load times.

2. **Code Splitting**: Use React's lazy loading and Suspense for code splitting.

3. **Caching**: Implement caching strategies using Redis or MongoDB's built-in caching.

4. **CDN**: Use a Content Delivery Network for static assets.

5. **Compression**: Enable Gzip compression in Nginx.

6. **Optimizing Database Queries**: Use indexing and limit the fields returned in queries.

7. **Load Balancing**: Implement load balancing for horizontal scaling.

8. **Minification**: Minify your JavaScript and CSS files.

9. **Lazy Loading**: Implement lazy loading for images and other media.

10. **API Optimization**: Use GraphQL or implement pagination for large datasets.

Remember to regularly profile your application's performance and make iterative improvements.

## Troubleshooting Common Issues

When deploying MERN stack applications, you might encounter some common issues. Here's how to address them:

1. **MongoDB Connection Issues**

   - Ensure MongoDB is running: `sudo systemctl status mongod`
   - Check MongoDB logs: `sudo tail -f /var/log/mongodb/mongod.log`
   - Verify connection string in your .env file

2. **Node.js Application Not Starting**

   - Check for syntax errors in your code
   - Ensure all dependencies are installed: `npm install`
   - Check PM2 logs: `pm2 logs api-prod`

3. **Nginx Configuration Issues**

   - Test Nginx configuration: `sudo nginx -t`
   - Check Nginx error logs: `sudo tail -f /var/log/nginx/error.log`

4. **SSL Certificate Problems**
   - Verify certificate renewal: `sudo certbot renew --dry-run`
   - Check SSL configuration in Nginx

