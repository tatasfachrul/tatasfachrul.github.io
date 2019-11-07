# Amazon Elastic Compute Cloud (EC2)

## Table of Content
1. Prerequiste
2. Setup Instance
3. Setup NodeJs
4. Setup NginX
5. Setup SSL (Letsencrypt)
6. Setup Docker

## 1. Requirement

1. Mempunyai AWS Account.
2. Mengerti perintah dasar Linux.
3. SSH Client (for Windows Only!)
4. Punya Domain

## 2. Setup Instance

`Follow the tutorial on EC2`

Connect to Instance with SSH Client
```bash
ssh -i credential.pem user@example.com
```

Update repository
```bash
apt-get update
```
## 3. Setup NodeJS

Install the latest NodeJS
```bash
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Create simple app with Express
```bash
mkdir simple-app
cd simple app
npm init -y
npm install express
```

Create app JS with direct `nano app.js` or `touch app.js` first or nano.


Create sample Endpoint
```js
const express = require('express')
const app = express()

app.listen(8001, () => { console.log('Server is Running' )})

app.get('/', (req, res) => { res.json('Hello world') })
```

Test your App
```bash
node app.js
```

Install process manager untuk NodeJS app
```bash
npm install -g pm2
```

Run your app
```bash
pm2 start app.js
```

Test your app at `http://your-domain.com:8001`


## 4. Setup NginX
```bash
npm install nginx
```

Create configuration for Reverse Proxy

`nano /etc/nginx/sites-available/your-domain.com.conf`

```bash
server {
  listen 80;

  server_name your-domain.com;

  location / {
    proxy_pass http://localhost:8001; # Change the port if needed
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}

```
`proxy_pass http://localhost:8001` is where your service is running on.

Now your app domain should looks like these `http://your-domain.com`

## 5. Setup SSL

We need more secure protocol, let's change from `HTTP` to `HTTPS` with free SSL from https://letsencrypt.org/

Prepare your certbot for your nginx
```bash
apt install python-certbot-nginx
```

You need to know that Let's Encrypt only give you 5 Attempt to create certificate. So we need to add `--dry-run` on our argument for staging mode. This will not used our attempt to create SSL. So we can debugging what's wrong when creating certificate.

Let's try staging mode to create SSL
```bash
certbot certonly --dry-run --nginx -d yourdomain.com
```

If this Success, then let's create the real deal.
```bash
certbot --nginx -d your-domain.com
```

Follow the instruction on the screen like
1. Requesting your email address
2. Terms of Service
3. Your engine (Apache or NginX)
4. Redirect HTTP to HTTPS

If success then let's proceed to the next step

Config your `sites-available/your-domain.com.conf` just like these.
```bash
server {
  listen 80;
  listen 443 ssl;

  server_name your-domain.com;

  ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
  include /etc/letsencrypt/options-ssl-nginx.conf;
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;


  location / {
    proxy_pass http://localhost:8001; # Change the port if needed
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}
```

From script above we need to tell the server that this service listening to encrypted protocol on `listen 443 ssl`, then we add bunch of certificate that genereted automatically with cert-bot.

Now restart your NginX
```bash
/etc/init.d/nginx restart
```

Then try access your app with `https` protocol `https://your-domain.com`

## 6. Setup Docker