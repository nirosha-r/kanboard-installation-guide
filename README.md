# Kanboard Installation & Configuration Guide

A step‑by‑step guide to installing, configuring, and customizing **Kanboard** with MySQL, plugins, themes, Gmail SMTP, HTTPS, and troubleshooting tips.

## 📖 What is Kanboard?
Kanboard is a free and open-source project management software that implemts the Kanban methodology. It helps teams visualize work, limit work-in-progress, and improve workflow efficiency.


## 📦 Prerequisites
- **OS:** CentOS/RHEL/Fedora or Ubuntu/Debian  
- **Webserver:** Apache or Nginx  
- **PHP:** ≥ 7.4 with extensions: `pdo_mysql`, `gd`, `mbstring`, `openssl`, `json`, `xml`  
- **Database:** MySQL (preferred)  
- **Git:** for cloning Kanboard and plugins  

---

## ⚙️ Installation
```sh
cd /var/www/html
sudo git clone https://github.com/kanboard/kanboard.git
cd kanboard
```

### Symlink Data & Plugins to `/srv`
```sh
sudo mkdir -p /srv/kanboard/data /srv/kanboard/plugins
sudo ln -s /srv/kanboard/data /var/www/html/kanboard/data
sudo ln -s /srv/kanboard/plugins /var/www/html/kanboard/plugins
```

### Permissions
```sh
sudo chown -R apache:apache /srv/kanboard
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/srv/kanboard(/.*)?"
sudo restorecon -Rv /srv/kanboard
```

---

## 🛠️ Configuration (`config.php`)

### MySQL Database
```php
define('DB_DRIVER', 'mysql');
define('DB_USERNAME', 'kanboard_user');
define('DB_PASSWORD', 'securepassword');
define('DB_HOSTNAME', '127.0.0.1');
define('DB_NAME', 'kanboard_db');
```

### General Settings
```php
// Enable plugin installer (optional)
define('PLUGIN_INSTALLER', true);

// Application URL
define('KANBOARD_URL', 'https://kanban.example.com');
```

---

## 🌐 Accessing Kanboard
Once installed and configured, open your browser:

```
http://your-server-ip/kanboard
```

or, if using a domain:

```
http://kanban.example.com
```

After HTTPS setup (see below):

```
https://kanban.example.com
```

---

## 🔒 HTTPS Setup with Let’s Encrypt (Apache)

### Install Certbot
```sh
sudo dnf install certbot python3-certbot-apache -y   # RHEL/Fedora
# OR
sudo apt install certbot python3-certbot-apache -y   # Debian/Ubuntu
```

### Request Certificate
```sh
sudo certbot --apache -d kanban.example.com
```

This will:
- Obtain a valid TLS certificate
- Configure Apache to redirect HTTP → HTTPS

### Auto‑Renewal
Certbot installs a systemd timer for auto‑renewal. Test with:
```sh
sudo certbot renew --dry-run
```

---

## 🎨 Plugins & Themes

### Install Plugins
```sh
cd /srv/kanboard/plugins
sudo git clone https://github.com/creecros/Customizer.git Customizer
sudo git clone https://github.com/kenlog/Moon.git Moon
sudo git clone https://github.com/kenlog/Nebula.git Nebula
sudo git clone https://github.com/Tagirijus/Darkboard.git Darkboard
```

Restart Apache:
```sh
sudo systemctl restart httpd
```

---

## 📧 Gmail SMTP Setup

### Generate App Password
1. Enable 2FA in Google Account.  
2. Create an **App Password** for “Mail”.  

### Configure `config.php`
```php
define('MAIL_TRANSPORT', 'smtp');
define('MAIL_SMTP_HOSTNAME', 'smtp.gmail.com');
define('MAIL_SMTP_PORT', 587);
define('MAIL_SMTP_ENCRYPTION', 'tls');
define('MAIL_SMTP_USERNAME', 'yourgmail@example.com');
define('MAIL_SMTP_PASSWORD', 'your-app-password');
define('MAIL_FROM', 'yourgmail@example.com');
define('MAIL_FROM_NAME', 'Project Kanban System');
```

### SELinux (if enabled)
```sh
sudo setsebool -P httpd_can_sendmail 1
```

### Test
- Go to **Admin → Settings → Email**.  
- Send a test email.  

---

## 🛠️ Troubleshooting

### Error: Missing `app.min.css` / `app.min.js`
**Symptom:**
```
Warning: filemtime(): stat failed for /var/www/html/kanboard/assets/css/app.min.css
```

**Solution:**
```sh
cd /var/www/html/kanboard
php cli/compress.php css
php cli/compress.php js
```

If `cli/compress.php` is missing, re‑clone the official repo:
```sh
sudo rm -rf /var/www/html/kanboard
sudo git clone https://github.com/kanboard/kanboard.git
```

---

### Error: “admin via Kanboard” in Emails
**Cause:** Default sender name not overridden.  
**Solution:** Add in `config.php`:
```php
define('MAIL_FROM_NAME', 'Project Kanban System');
```

---

### Error: “Plugin Directory … Not Configured”
**Cause:** Kanboard doesn’t connect to an online marketplace by default.  
**Solution:** Manual plugin installation is the standard method. Ignore this message unless you want to enable `PLUGIN_INSTALLER`.

---

## 🔄 Maintenance

### Update Kanboard
```sh
cd /var/www/html/kanboard
sudo git pull origin master
```

### Update Plugins
```sh
cd /srv/kanboard/plugins/Customizer
sudo git pull
```

---

## ✅ Summary
This guide covers:
- Clean installation with `/srv` separation  
- **MySQL database configuration**  
- Browser access and HTTPS setup with Let’s Encrypt  
- Plugin & theme setup (Customizer, Moon, Nebula, Darkboard)  
- Troubleshooting common errors (`app.min.css` missing, “admin via Kanboard”, plugin directory message)  
- Gmail SMTP configuration for notifications

## 📂 Data & Plugins Location
Kanboard needs writable directories for data (database, attachments) and plugins.
Instead of keeping them inside /var/www/html/kanboard, this guide places them in /srv/kanboard:
- Security → keeps sensitive files outside the webroot
- Upgrades → git pull won’t overwrite your data/plugins
- Backups → easier to archive separately

### Alternatives
- You can use any other partition (e.g. /data/kanboard, /mnt/storage/kanboard) — just adjust the symlinks and permissions accordingly.
---

Maintained by Nirosha-R
