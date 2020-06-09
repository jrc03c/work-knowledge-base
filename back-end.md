# NGINX w/ Express

> **NOTE:** The following steps assume that you'll be running NGINX as a reverse proxy for an Express app (on Ubuntu 18.04 LTS).

**1.** Install NGINX:

```
sudo apt-get install nginx
```

**2.** Install [Certbot](https://certbot.eff.org/):

```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot python3-certbot-nginx
```

**3.** Create and configure a server block in `/etc/nginx/sites-available/example.com`:

```
server {
  server_name example.com;
}
```

**4.** Symlink the newly-created configuration file into `/etc/nginx/sites-enabled` and delete the default file in the same folder:

```
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
sudo rm /etc/nginx/sites-enabled/default
```

**5.** Confirm that the configuration is valid:

```
sudo nginx -t
```

**6.** Restart NGINX:

```
sudo service nginx restart
```

**7.** Request Certbot certificates. Note that this will modify the NGINX configuration files for any selected domains (e.g., `/etc/nginx/sites-available/example.com`).

```
sudo certbot --nginx
```

**8.** Restart NGINX:

```
sudo service nginx restart
```

**9.** Build a simple Express app (or use a preexisting one) that'll run on port 8000:

```js
let express = require("express")
let app = express()

app.get("/", function(request, response){
  return response.send("Hello, world!")
})

app.listen(8000)
```

**9.** Start the Express app.

```
forever start /path/to/app.js
```

**10.** Modify `/etc/nginx/sites-available/example.com` such that the SSL section (the server block instructed to listen on port 443) should have a new feature:

```
server {
  listen 443 ssl;
  ...
  
  location / {
    proxy_pass http://127.0.0.1:8000;
  }
}

server {
  listen 80;
  ...
}
```

**11.** Confirm that the NGINX configuration is valid:

```
sudo nginx -t
```

**12.** Restart NGINX:

```
sudo service nginx restart
```

**13.** Confirm that you can visit https://example.com in the browser!
