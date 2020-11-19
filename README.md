# Node.js Deployment

> Steps to deploy a Node.js app to your own (home) server using PM2, NGINX as a reverse proxy and an SSL from LetsEncrypt

## 1. Open port 80 on your router
You might need to do this through your internet provider. Optimum has that function on their website in your profile.

## 2. Port Forward port 80 (http) on your router to port 80 on your server IP and same for port 443 (https)
 You can do this by accessing your router settings through a browser.

## 3. Install Node/NPM on your server computer
```
curl -sL https://deb.nodesource.com/setup_15.x | sudo -E bash -

sudo apt install nodejs

node --version
```

## 4. Clone your project from Github
Clone your project. Here is a simple node.js server:
https://github.com/nvoevodin/LetsEncrypt_Documentation.git

It is a hello world server running on port 5000


```
git clone https://github.com/nvoevodin/LetsEncrypt_Documentation.git #clone


```



### 5. Install dependencies and test app
```
npm install #install dependencies

node index.js #start the server

```
Now, go to the localhost:5000 in your browser to see 'hello world'

## 6. Setup PM2 process manager to keep your app running
```
sudo npm i pm2 -g
pm2 start app (or whatever your file name)

# Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs (Show log stream)
pm2 flush (Clear logs)

# To make sure app starts when reboot
pm2 startup ubuntu
```
### You should now be able to access your app using your IP and port. Now we want to setup a firewall blocking that port and setup NGINX as a reverse proxy so we can access it directly using port 80 (http)

## 7. Setup ufw firewall
```
sudo ufw enable
sudo ufw status
sudo ufw allow ssh (Port 22)
sudo ufw allow http (Port 80)
sudo ufw allow https (Port 443)
```

## 8. Install NGINX and configure
```
sudo apt install nginx

sudo nano /etc/nginx/sites-available/default # this is the settings file
```
Add the following to the location part of the server block
```
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:5000; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```
```
# Check NGINX config
sudo nginx -t # This checks if the config file is all good

# Restart NGINX
sudo service nginx restart
```

### You should now be able to visit your IP with no port (port 80) and see your app. Now let's add a domain

## 9. Add domain in godaddy
Buy a domain

Point the A record to your public IP (just IP address and nothing else after it)

You can find your public ip by typing 'my public ip' in google.

It is different from your usual 192.168.xxx.xx.

It may take a bit to propogate (usually 10-30 mins)

If you were able to see your hello world by visiting your localhost before, you should be able to do the same with your public ip as well. Once godaddy links your domain to that public ip, you should be able to see your hello world by visitin your domain.

10. Add SSL with LetsEncrypt
```
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Only valid for 90 days, test the renewal process with
certbot renew --dry-run #using their test server (no renew is really happening)
```

Now visit https://yourdomain.com and you should see your Node app