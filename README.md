# Instalasi Odoo 17 pada Ubuntu 22.04.3 LTS (Jammy Jellyfish)

## Bagian I -> [Unbuntu Prep](ubuntu-prep.md)

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
Atau bisa hanya di level database saja
```
GRANT ALL PRIVILEGES ON DATABASE odoo17 TO odoo17;
```

Keluar dari terminal PSQL
```
\q
```

Dilanjutkan keluar dari terminal PostgreSQL
```
exit
```

Setup tambahan ketika install PostgreSQL di OCI (Oracle Cloud Infrastructure)
```
sudo apt-get install iptables-persistent
```

Add rule iptables
```
sudo iptables -I INPUT 5 -p tcp --dport 5432 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
```

Lalu simpan supaya perubahan ini permanen
```
sudo netfilter-persistent save
```

Restart service postgres
```
sudo systemctl restart postgresql
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
su odoo17
```
Pindah ke direktori kerja
```
cd /opt/odoo17
```
Clone repository dari GitHub
```
git clone --branch=17.0 --depth=1 --single-branch "https://www.github.com/odoo/odoo" odoo17
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
nano /etc/odoo17.conf
```
Jangan lupa mengganti password sesuai dengan user.
```
[options]
    admin_passwd = mypassword
    db_host = False
    db_port = False
    db_user = odoo17
    db_password = False
    dbfilter = %h
    addons_path = /opt/odoo17/odoo17/addons, /opt/odoo17/odoo17/custom-addons

    log_db = False
    log_db_level = warning
    log_handler = :INFO
    log_level = info
    logfile = /var/log/odoo17/odoo.log

    proxy_mode = True

    web.base.url = "https://yourdomain.id"
    report.url = "http://localhost:8069"
    report.url.freeze = True
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
chown odoo17: /etc/odoo17.conf
```
```
chmod 640 /etc/odoo17.conf
```

Reload config
```
systemctl daemon-reload
```
```
systemctl enable --now odoo17
```

## Bagian VIII: SSL Certificate

Nginx diperlukan untuk install dan aktivasi sertifikat SSL.
```
apt-get install nginx -y
```

Install certbot.
```
apt-get install certbot python3-certbot-nginx -y
```

Jalankan certbot untuk request sertifikat.

Command untuk single domain (automatic renewal)
```
certbot --nginx -d mydomain.id -m mail@mydomain.id --agree-tos
```

Command untuk wildcard SSL (manual renewal)
```
certbot certonly --manual \
--preferred-challenges=dns \
--email admin@mydomain.id \
--server https://acme-v02.api.letsencrypt.org/directory \
--agree-tos \
--manual-public-ip-logging-ok \
-d mydomain.id, *.mydomain.id
```

@location of the cert.
```
/etc/letsencrypt/live/mydomain.id/fullchain.pem
/etc/letsencrypt/live/mydomain.id/privkey.pem
```

Auto renew certificates
```
crontab -e
```
Paste kode cron, jalan setiap hari.
```
0 12 * * * /usr/bin/certbot renew --quiet
```
Simpan dan exit.

## Bagian IX: Secure with SSL
```
cd /etc/nginx/sites-available
```
Buat konfigurasi
```
nano odoo-nginx.conf
```
Lalu paste file yang ada pada repository ini (nginx.conf)
Save
```
sudo ln -s /etc/nginx/sites-available/odoo-nginx.conf /etc/nginx/sites-enabled/odoo-nginx.conf
```
Unlink default config
```
unlink /etc/nginx/sites-enabled/default
```
Cek ulang konfigurasi
```
nginx -t
```
Jika ok, lanjut apply konfigurasi baru
```
service nginx restart
```

## Final
Default port for Odoo
```
http://host:8069
```