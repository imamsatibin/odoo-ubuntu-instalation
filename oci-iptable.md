## Oracle Cloud Infrastructure (OCI)

Pada platform ini selain mendaftarkan port yang ingin dibuka pada VCN. Perlu juga mengganti `iptable` pada instance. 

Edit pada file ini `/etc/iptables/rules.v4`
```
sudo nano /etc/iptables/rules.v4
```

Tambahkan port lain (misal 80, 443), copy saja dari `port 22` yang sudah terbuka secara default.
Jangan lupa SAVE dan EXIT.

Apply perubahan dengan.
```
sudo iptables-restore < /etc/iptables/rules.v4
```

Jika OK semua, kadang ada baiknya coba restart instance, untuk memastikan saja.