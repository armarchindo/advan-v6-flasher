# advan-v6-flasher
Flasher firmware MOD untuk modem CPE Advan V6 atau ZLT S50
PRASYARAT WAJIB AGAR INSTALL SUKSES, BACA DULU INI
```
fb.com/share/p/1CDswtfsiY/
```

Cara install urut:
1. Download aplikasi bernama Termux di playstore
2. Pastikan Modem Advan V6 punya koneksi Internet dengan kuota minimal 500MB
3. Koneksikan HP Android (wajib) ke wifi Advan V6
4. Copas dan Enter command ini ke termux
```
pkg update && pkg upgrade -y && pkg install wget && wget https://raw.githubusercontent.com/armarchindo/advan-v6-flasher/main/advanv6_flasher -O advanv6_flasher && chmod +x ./advanv6_flasher && ./advanv6_flasher
```
5. Next (ENTER) sampai selesai dan tunggu modem restart

Detail cara untuk memakai firmware ini bisa dilihat di grup FB IHABT
```
fb.com/share/p/1CDswtfsiY/
```

OPSIONAL!!! TIDAK WAJIB
untuk pengaktivasi TELNET dan SSH gunakan aplikasi ini
```
pkg update && pkg upgrade -y && pkg install wget && wget https://raw.githubusercontent.com/armarchindo/advan-v6-flasher/main/advanv6_activator -O advanv6_activator && chmod +x ./advanv6_activator && ./advanv6_activator
```
