# NGINX w/ Express

> **NOTE:** The following steps assume that you'll be running `nginx` as a reverse proxy for an Express app.

**1.** Install NGINX:

```bash
sudo apt-get install nginx
```

**2.** Install [Certbot](https://certbot.eff.org/).

**3.** Request Certbot certificates manually (if the DNS records aren't set yet). This method requires the creation of TXT DNS records. (Don't forget to add the `--dry-run` flag for testing first!)

```bash
sudo certbot certonly --manual --preferred-challenges=dns -d example.com
```

**4.** Create `/etc/nginx/sites-available/example.com` and fill with:

```
server {
  listen 80;
  server_name example.com;
  return 301 https://example.com$request_uri;
}

server {
  listen 443 ssl;
  server_name example.com;

  ssl on;

  ssl_certificate /etc/letsencrypt/live/example.com/cert.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

  ssl_stapling on;
  ssl_stapling_verify on;
  ssl_trusted_certificate /etc/letsencrypt/live/example.com/fullchain.pem;

  ssl_session_timeout 5m;

  location / {
    proxy_pass http://127.0.0.1:8000;
  }
}
```

Note that this does two things. First, it redirects all HTTP traffic to HTTPS. Second, it sends all HTTPS traffic to the Express app running on port 8000.

**5.** Symlink this new configuration file to `/etc/nginx/sites-enabled` with:

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
```

**6.** Test the NGINX configuration with:

```bash
sudo nginx -t
```

**7.** If it passes, restart the NGINX service with:

```bash
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

**10.** Start the Express app.

```bash
forever start /path/to/app.js
```

**11.** Confirm that you can access https://example.com in the browser.
