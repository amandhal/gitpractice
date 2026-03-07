# 📘 Web Servers -- Day 1 (NGINX Basics)

## 📌 Objective

Learn the fundamentals of **NGINX web server configuration**, including:

-   Installing NGINX
-   Understanding configuration structure
-   Creating a simple web server
-   Handling requests and responses

------------------------------------------------------------------------

# 🧠 Key Concepts Covered

-   Web servers
-   NGINX architecture
-   `server` blocks
-   `location` blocks
-   Default server behaviour
-   Request routing

------------------------------------------------------------------------

# ⚙️ Environment Setup

### Install NGINX

``` bash
sudo apt update
sudo apt install nginx
```

### Verify Installation

``` bash
nginx -v
```

Start NGINX

``` bash
sudo systemctl start nginx
```

Enable on boot

``` bash
sudo systemctl enable nginx
```

Check status

``` bash
sudo systemctl status nginx
```

------------------------------------------------------------------------

# 📂 NGINX Important Directories

  Path                           Purpose
  ------------------------------ -----------------------
  `/etc/nginx/nginx.conf`        Main configuration
  `/etc/nginx/sites-available`   Virtual host configs
  `/etc/nginx/sites-enabled`     Enabled sites
  `/var/www/html`                Default web root
  `/var/log/nginx`               Access and error logs

------------------------------------------------------------------------

# 🧾 Basic NGINX Server Configuration

Example configuration:

``` nginx
server {
    listen 80;
    server_name mysite.local;

    root /var/www/mysite.local/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Explanation

  Directive       Meaning
  --------------- -----------------------
  `listen 80`     Listen on HTTP port
  `server_name`   Domain name
  `root`          Website directory
  `index`         Default file
  `try_files`     Checks requested file

------------------------------------------------------------------------

# 📁 Create Website Directory

``` bash
sudo mkdir -p /var/www/mysite.local/html
```

Set ownership

``` bash
sudo chown -R $USER:$USER /var/www/mysite.local
```

Create test page

``` bash
nano /var/www/mysite.local/html/index.html
```

Example HTML:

``` html
<h1>Welcome to My NGINX Server</h1>
```

------------------------------------------------------------------------

# 🔗 Enable Site

Create config file

``` bash
sudo nano /etc/nginx/sites-available/mysite.local
```

Enable site

``` bash
sudo ln -s /etc/nginx/sites-available/mysite.local /etc/nginx/sites-enabled/
```

Test configuration

``` bash
sudo nginx -t
```

Reload NGINX

``` bash
sudo systemctl reload nginx
```

------------------------------------------------------------------------

# 🧪 Testing

Open browser:

    http://mysite.local

Or using curl:

``` bash
curl localhost
```

------------------------------------------------------------------------

# 🚨 Common Errors

### 403 Forbidden

Directory permission issue.

Fix:

``` bash
chmod -R 755 /var/www
```

------------------------------------------------------------------------

### 404 Not Found

File does not exist or wrong `root` path.

------------------------------------------------------------------------

# 📚 What You Learned

-   Installing NGINX
-   Creating server blocks
-   Setting document root
-   Enabling virtual hosts
-   Testing configuration

------------------------------------------------------------------------

# 📅 Training Series

  Day     Topic
  ------- ----------------
  Day 1   NGINX Basics
  Day 2   NGINX Routing
  Day 3   Reverse Proxy
  Day 4   Load Balancing

------------------------------------------------------------------------

# 🚀 CloudMaven DevOps Internship

Hands-on DevOps training covering Web Servers, CI/CD, Containers, and
Cloud.
