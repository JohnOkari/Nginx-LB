---

# 🚀 Nginx Load Balancer with SSL/TLS (HTTPS)

This project demonstrates how to configure **Nginx as a Load Balancer** for multiple web servers and secure the traffic using **SSL/TLS certificates** from [Let’s Encrypt](https://letsencrypt.org/) via **Certbot**.

## The final architecture looks like this:

# ![Architecture](https://steghub.com/wp-content/uploads/2023/07/image-42-1024x623.png)

---

## 📌 Prerequisites

* AWS EC2 instance (Ubuntu 20.04 LTS) for **Nginx Load Balancer**
* At least **two backend web servers** (e.g., Apache or Nginx running a sample app)
* A registered domain name (from GoDaddy, Namecheap, Bluehost, etc.)
* Elastic IP attached to your EC2 instance (for static IP mapping)
* Ports **80 (HTTP)** and **443 (HTTPS)** opened in **Security Groups**

---

## ⚙️ Part 1: Configure Nginx as a Load Balancer

### 1️⃣ Launch an EC2 Instance

* Create a new **Ubuntu 20.04 LTS** instance.
* Open **TCP Port 80** (HTTP) and **443** (HTTPS) in Security Groups.
* (Optional) Terminate your old Apache LB instance if previously configured.

### 2️⃣ Update `/etc/hosts` File

Add entries for backend web servers (Web1 & Web2) using their **private IPs**:

```bash
sudo vi /etc/hosts
```

Example:

```
10.0.1.10   Web1
10.0.1.11   Web2
```

### 3️⃣ Install Nginx

```bash
sudo apt update
sudo apt install nginx -y
```

### 4️⃣ Configure Load Balancing in Nginx

Edit Nginx config file:

```bash
sudo vi /etc/nginx/nginx.conf
```

Inside the **http** block, add:

```nginx
upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
}

server {
    listen 80;
    server_name www.domain.com;

    location / {
        proxy_pass http://myproject;
    }
}
```

➡️ Comment out:

```nginx
#include /etc/nginx/sites-enabled/*;
```

### 5️⃣ Restart Nginx

```bash
sudo systemctl restart nginx
sudo systemctl status nginx
```

✅ At this point, traffic is distributed between Web1 and Web2 via HTTP.

---

## 🔒 Part 2: Enable SSL/TLS with Let’s Encrypt

### 1️⃣ Register a Domain Name

* Purchase/register a domain name (e.g., **example.com**) from a domain registrar.

### 2️⃣ Assign Elastic IP to EC2

* Allocate an **Elastic IP** in AWS.
* Associate it with the Nginx LB instance.
* Update your domain’s **A Record** to point to the Elastic IP.

Check propagation:

```bash
ping your-domain.com
```

### 3️⃣ Update Nginx Config with Domain Name

Edit `/etc/nginx/nginx.conf` again and update `server_name`:

```nginx
server {
    listen 80;
    server_name www.your-domain.com;

    location / {
        proxy_pass http://myproject;
    }
}
```

### 4️⃣ Install Certbot (for SSL Certificates)

Ensure `snapd` is running:

```bash
sudo systemctl status snapd
```

Install Certbot:

```bash
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### 5️⃣ Request SSL Certificate

```bash
sudo certbot --nginx
```

* Select your domain from the prompt.
* Certbot automatically updates Nginx with SSL configs.

### 6️⃣ Test HTTPS

Visit:

```
https://your-domain.com
```

✅ You should see a **padlock icon** in the browser (certificate valid).

---

## 🔄 Part 3: Auto Renewal of SSL Certificates

Let’s Encrypt certificates are valid for **90 days**. Set up automatic renewal.

### 1️⃣ Test Renewal Command

```bash
sudo certbot renew --dry-run
```

### 2️⃣ Setup Cron Job for Renewal

Edit crontab:

```bash
sudo crontab -e
```

Add:

```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

➡️ This runs twice a day.

---

## ✅ Verification

* **HTTP** → Redirects to HTTPS
* **HTTPS** → Encrypted (port 443)
* Load balancing works (traffic distributed between Web1 & Web2)
* SSL certificate is auto-renewed

---

## 📚 Useful References

* [Nginx Load Balancing](https://nginx.org/en/docs/http/load_balancing.html)
* [Let’s Encrypt & Certbot](https://certbot.eff.org/)
* [AWS Elastic IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)
* [DNS Records Explained](https://www.cloudflare.com/learning/dns/dns-records/)

---

## 🎉 Conclusion

You have successfully configured:

* An **Nginx Load Balancer** for multiple web servers
* **SSL/TLS encryption** using Let’s Encrypt (Certbot)
* **Auto-renewal** of SSL certificates with cron

This ensures your application is **scalable, secure, and production-ready**.

---