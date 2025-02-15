## Bagian I: Initial Setup

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
timedatectl set-timezone Asia/Singapore
```

Amankan server (proteksi layer tambahan)
```
apt-get install openssh-server fail2ban -y
```

## Bagian I: Tambah User (for daily use)

Run command ini untuk menambahkan user baru.
Misal usernya `imam`
```
adduser imam
```

Hit/tekan `ENTER` dan ikuti requirement sistem.

Berikan akses sudo/administrative task
```
usermod -aG sudo imam
```

Salin SSH Key root supaya next time kita login menggunakan SSH Key bukan plain password biasa
```
rsync --archive --chown=imam:imam ~/.ssh /home/imam
```