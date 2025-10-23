
# ⚙️ DirectAdmin Laravel Symlink Hook

Custom DirectAdmin **post-create hook** yang secara otomatis membuat **symlink** dari `public_html` ke `laravel/public` untuk domain tertentu (contohnya `*.domainkamu.com`).  
Berguna ketika kamu menjalankan banyak proyek Laravel di bawah DirectAdmin, agar struktur domain lebih konsisten dan siap digunakan tanpa konfigurasi manual.

---

## 🚀 Fitur

✅ Otomatis membuat **symlink** `public_html → laravel/public`  
✅ Hanya berlaku untuk domain dengan pola tertentu (misal: `domainkamu.com`)  
✅ Membuat folder `laravel/public` jika belum ada  
✅ Logging lengkap di `/var/log/directadmin/domain_symlink.log`  
✅ Aman, tidak mengganggu domain lain  

---

## 🧩 Struktur File

```

/usr/local/directadmin/scripts/custom/domain_create_post.sh
/var/log/directadmin/domain_symlink.log

````

---

## 🛠️ Instalasi

1. **Masuk ke server DirectAdmin**
   ```bash
   ssh root@yourserver
   ```

2. **Buat folder custom hook jika belum ada**

   ```bash
   mkdir -p /usr/local/directadmin/scripts/custom
   ```

3. **Buat file hook baru**

   ```bash
   nano /usr/local/directadmin/scripts/custom/domain_create_post.sh
   ```

4. **Paste isi script berikut:**

   ```bash
   #!/bin/bash
   # Hook: domain_create_post.sh
   # Purpose: Symlink public_html ke laravel/public untuk domain domainkamu.com

   LOG_FILE="/var/log/directadmin/domain_symlink.log"

   DOMAIN="$domain"
   USER="$username"
   HOMEDIR="/home/$USER/domains/$DOMAIN"
   TARGET_LARAVEL="$HOMEDIR/laravel/public"
   PUBLIC_HTML="$HOMEDIR/public_html"

   echo "[$(date '+%Y-%m-%d %H:%M:%S')] Domain creation detected: $DOMAIN for user $USER" >> $LOG_FILE

   if [[ "$DOMAIN" == *"domainkamu.com" ]]; then
       echo "[$(date '+%Y-%m-%d %H:%M:%S')] Domain $DOMAIN matched domainkamu.com pattern" >> $LOG_FILE

       if [ ! -d "$TARGET_LARAVEL" ]; then
           echo "[$(date '+%Y-%m-%d %H:%M:%S')] Folder $TARGET_LARAVEL not found, creating..." >> $LOG_FILE
           mkdir -p "$TARGET_LARAVEL"
           chown -R "$USER:$USER" "$HOMEDIR/laravel"
       fi

       if [ -d "$PUBLIC_HTML" ] && [ ! -L "$PUBLIC_HTML" ]; then
           echo "[$(date '+%Y-%m-%d %H:%M:%S')] Removing default public_html directory..." >> $LOG_FILE
           rm -rf "$PUBLIC_HTML"
       fi

       if [ ! -L "$PUBLIC_HTML" ]; then
           ln -s "$TARGET_LARAVEL" "$PUBLIC_HTML"
           chown -h "$USER:$USER" "$PUBLIC_HTML"
           echo "[$(date '+%Y-%m-%d %H:%M:%S')] Created symlink: $PUBLIC_HTML -> $TARGET_LARAVEL" >> $LOG_FILE
       else
           echo "[$(date '+%Y-%m-%d %H:%M:%S')] Symlink already exists for $PUBLIC_HTML" >> $LOG_FILE
       fi
   else
       echo "[$(date '+%Y-%m-%d %H:%M:%S')] Domain $DOMAIN skipped (not domainkamu.com)" >> $LOG_FILE
   fi

   exit 0
   ```

5. **Beri izin eksekusi**

   ```bash
   chmod 755 /usr/local/directadmin/scripts/custom/domain_create_post.sh
   ```

6. **Tes manual (opsional)**

   ```bash
   domain="test.domainkamu.com" username="ahmad" /usr/local/directadmin/scripts/custom/domain_create_post.sh
   ```

7. **Cek hasil log**

   ```bash
   tail -n 20 /var/log/directadmin/domain_symlink.log
   ```

---

## 📄 Contoh Log Output

```
[2025-10-23 09:30:12] Domain creation detected: api.domainkamu.com for user ahmad
[2025-10-23 09:30:12] Domain api.domainkamu.com matched domainkamu.com pattern
[2025-10-23 09:30:12] Folder /home/ahmad/domains/api.domainkamu.com/laravel/public not found, creating...
[2025-10-23 09:30:12] Removing default public_html directory...
[2025-10-23 09:30:12] Created symlink: /home/ahmad/domains/api.domainkamu.com/public_html -> /home/ahmad/domains/api.domainkamu.com/laravel/public
```

---

## 🔍 Catatan Tambahan

* Script ini **tidak akan dijalankan** untuk domain selain `domainkamu.com`.
* Untuk memperluas ke domain lain, ubah pola di bagian:

  ```bash
  if [[ "$DOMAIN" == *"domainkamu.com" ]]; then
  ```
* Untuk hook lain (misal `domain_restore_post.sh` atau `domain_change_post.sh`), kamu bisa gunakan script serupa agar symlink tetap konsisten setelah restore/clone.

---

## 📦 Lisensi

MIT License © 2025 Ahmad Fajrul Falaah
Feel free to fork, modify, and use for your DirectAdmin + Laravel deployments.

---

## 🧠 Credits

* Original implementation by [Ahmad Fajrul Falaah](https://github.com/alfalaah404)
