# Instalasi XRay pada Ubuntu 22.04.3 LTS (Jammy Jellyfish)

## Bagian I: Ubuntu Prep

Pastikan dulu jalankan sebagai sudoer.
```
sudo su
```

Lalu, cek update OS.
```
apt-get update
```

Jalankan command upgrade.
```
apt-get upgrade -y
```

Jangan lupa, set timezone (jika belum).
```
timedatectl set-timezone Asia/Jakarta
```

Amankan server (proteksi layer tambahan)
```
apt-get install openssh-server fail2ban
```

## Bagian II: Packages and Libraries
```
apt-get install python3-pip -y
```
```
apt-get install -y git python3-venv python3-dev libxml2-dev libxslt1-dev zlib1g-dev libsasl2-dev libldap2-dev build-essential libssl-dev libffi-dev libmysqlclient-dev libjpeg-dev libpq-dev libjpeg8-dev liblcms2-dev libblas-dev libatlas-base-dev
```
```
apt-get install npm -y
```
```
ln -s /usr/bin/nodejs /usr/bin/node
```
```
npm install -g less less-plugin-clean-css
```
```
apt-get install -y node-less
```

## Bagian III: Instalasi dan Konfigurasi PostgreSQL
Langkah pertama menginstall PostgreSQL
```
apt-get install postgresql -y
```
Pindah ke dalam terminal PostgreSQL
```
su - postgres
```
Buat user database dengan nama odoo17
```
createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo17
```
Setelah command tersebut di atas di-execute, kita akan diarahkan untuk mengisi password, ingat dan simpan, karena akan digunakan sebagai config nantinya.

Masuk ke terminal PSQL (di dalam PostgreSQL)
```
psql
```
Berikan hak akses user odoo17 sebagai super user.
```
ALTER USER odoo17 WITH SUPERUSER;
```

Keluar dari terminal PSQL
```
\q
```

Dilanjutkan keluar dari terminal PostgreSQL
```
exit
```

## Bagian IV: Instalasi wkhtmltopdf
```
apt-get install wkhtmltopdf -y
```
Jika ditemukan ada error, jalankan perintah ini.
```
apt --fix-broken install
```

## Bagian V: Buat user Odoo (System Ubuntu)
```
useradd -m -d /opt/odoo17 -U -r -s /bin/bash odoo17
```

## Bagian VI: Install Odoo
Login sebagai user odoo17
```
su – odoo17
```
Pindah ke direktori kerja
```
cd ~/opt/odoo17
```
Clone repository dari GitHub
```
git clone --branch=17.0 --depth=1 --single-branch "https://www.github.com/odoo/odoo"
```
Proses instalasi menggunakan Phyton environtment
```
python3 -m venv odoo17-venv
```
```
source odoo17-venv/bin/activate
```
```
pip3 install wheel
```
```
pip3 install -r odoo17/requirements.txt
```
```
deactivate
```

Buat direktori untuk Custom Addon
```
mkdir /opt/odoo17/odoo17/custom-addons
```
Keluar dari user odoo17
```
exit
```

## Bagian VII: Konfigurasi Odoo, Start-up
Buat direktori untuk menampung log error
```
sudo mkdir /var/log/odoo17
```
```
sudo chown odoo17: /var/log/odoo17
```

Lanjut dengan membuat file konfig
```
nano /etc/odoo.conf
```
Jangan lupa mengganti password sesuai dengan user.
```
[options]
    admin_passwd = mypassword
    db_host = False
    db_port = False
    db_user = odoo17
    db_password = False
    addons_path = /opt/odoo17/odoo17/addons, /opt/odoo17/odoo17/custom-addons

    log_db = False
    log_db_level = warning
    log_handler = :INFO
    log_level = info
    logfile = /var/log/odoo17/odoo.log

    proxy_mode = True
```
Buat service, supaya auto-run ketika system reboot
```
nano /etc/systemd/system/odoo17.service
```
```
[Unit]
Description=odoo17
Requires=postgresql.service
After=network.target postgresql.service
[Service]
Type=simple
SyslogIdentifier=odoo17
PermissionsStartOnly=true
User=odoo17
Group=odoo17
ExecStart=/opt/odoo17/odoo17-venv/bin/python3 /opt/odoo17/odoo17/odoo-bin -c /etc/odoo17.conf
StandardOutput=journal+console
[Install]
WantedBy=multi-user.target
```

Grant akses kepada odoo17
```
chown odoo17: /etc/odoo.conf
```
```
chmod 640 /etc/odoo.conf
```

Reload config
```
systemctl daemon-reload
```
```
systemctl enable --now odoo17
```

## Bagian II: SSL Certificate

Nginx diperlukan untuk install dan aktivasi sertifikat SSL.
```
apt-get install nginx
```

Install certbot.
```
apt-get install certbot python3-certbot-nginx -y
```

Jalankan certbot untuk request sertifikat.
```
certbot --nginx -d mydomain.id -m mail@mydomain.id --agree-tos
```

@location of the cert.
```
/etc/letsencrypt/live/mydomain.id/fullchain.pem
/etc/letsencrypt/live/mydomain.id/privkey.pem
```

## Bagian III: Secure with SSL
