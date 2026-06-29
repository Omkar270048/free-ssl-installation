# free-ssl-installation
Here’s a **simple clean step-by-step SSL setup doc** for your setup (Django + Docker + EC2 + GoDaddy + Nginx).

---

# 🔒 Free SSL Setup (Let’s Encrypt) — Django on AWS (Docker)

## 🧱 Prerequisites

* EC2 Ubuntu server running
* Django running in Docker on port `8000`
* Domain (e.g. `your_domain.in`) from GoDaddy pointing to EC2 IP
* Nginx installed

---

# 1️⃣ Point Domain to EC2 (GoDaddy DNS)

Add these records in GoDaddy:

* **A Record**

  * Host: `@`
  * Value: `EC2_PUBLIC_IP`

* **A Record**

  * Host: `www`
  * Value: `EC2_PUBLIC_IP`

Wait 5–10 minutes.

---

# 2️⃣ Run Django Docker on port 8000

In `docker-compose.yml`:

```yaml id="d1"
ports:
  - "8000:8000"
```

Restart:

```bash id="d2"
docker compose down
docker compose up -d --build
```

---

# 3️⃣ Install Nginx

```bash id="d3"
sudo apt update
sudo apt install nginx -y
```

---

# 4️⃣ Configure Nginx

```bash id="d4"
sudo nano /etc/nginx/sites-available/django
```

Paste:

```nginx id="d5"
server {
    listen 80;
    server_name your_domain.in www.your_domain.in;

    location / {
        proxy_pass http://127.0.0.1:8000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable it:

```bash id="d6"
sudo ln -s /etc/nginx/sites-available/django /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

# 5️⃣ Open AWS Ports

In Security Group:

* ✔ HTTP → 80
* ✔ HTTPS → 443

---

# 6️⃣ Install Certbot (SSL tool)

```bash id="d7"
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

---

# 7️⃣ Get SSL Certificate

```bash id="d8"
sudo certbot --nginx
```

During setup:

* Enter email
* Select `your_domain.in` and `www.your_domain.in`
* Choose **Redirect HTTP → HTTPS (YES)**

---

# 🎉 DONE

Now your site is live:

```text id="d9"
https://your_domain.in
```

---

# 🔄 Auto Renewal (IMPORTANT)

Test:

```bash id="d10"
sudo certbot renew --dry-run
```

---

# 🧠 Final Flow

```text id="d11"
Browser
  ↓
HTTPS (443)
  ↓
Nginx (EC2)
  ↓
Docker Django (8000)
  ↓
MySQL container
```

---

# ⚠️ Key Points

* Don’t expose Django on port 80
* Only Nginx handles 80/443
* Docker stays internal on 8000
* SSL auto-renews every 90 days

---

If you want, I can next give you a **“production upgrade checklist”** (Gunicorn, caching, security hardening, CI/CD).

