
---

# 📘 ** Configuration LVM et migration MariaDB vers `/data`

## 1. Préparation du disque et création du LVM

1. Vérifier les disques présents :

   ```bash
   lsblk
   df -h
   ```

2. Créer une partition LVM sur le disque `/dev/vdb` :

   ```bash
   fdisk /dev/vdb
   ```

   * `n` → nouvelle partition
   * `p` → partition primaire
   * Enter → utiliser tout le disque
   * `t` → type `8e` (Linux LVM)
   * `w` → écrire et quitter

3. Initialiser le PV :

   ```bash
   pvcreate /dev/vdb1
   ```

4. Créer le VG :

   ```bash
   vgcreate data /dev/vdb1
   ```

5. Créer le LV :

   ```bash
   lvcreate -n data -l +100%FREE data
   ```

6. Vérifier :

   ```bash
   lvs
   ```

7. Formater en ext4 :

   ```bash
   mkfs.ext4 /dev/data/data
   ```

8. Créer le point de montage :

   ```bash
   mkdir /data
   ```

9. Monter temporairement pour tester :

   ```bash
   mount /dev/data/data /data
   ```

10. Vérifier :

```bash
df -h | grep /data
```

11. Ajouter dans `/etc/fstab` :

```fstab
/dev/mapper/data-data   /data   ext4   defaults   0   2
```

12. Tester avant reboot :

```bash
umount /data
mount -a
df -h | grep /data
```

---

## 2. Migration de MariaDB vers `/data`

1. Stopper MariaDB :

   ```bash
   systemctl stop mysql
   ```

2. Sauvegarder au cas où :

   ```bash
   cp -a /var/lib/mysql /root/mysql_backup
   ```

3. Déplacer le répertoire :

   ```bash
   mv /var/lib/mysql /data/
   ```

4. Créer le symlink :

   ```bash
   ln -s /data/mysql /var/lib/mysql
   ```

5. Vérifier :

   ```bash
   ls -l /var/lib/ | grep mysql
   ```

6. Vérifier les permissions :

   ```bash
   chown -R mysql:mysql /data/mysql
   ```

---

## 3. Configuration d’un répertoire temporaire dédié

1. Créer le répertoire :

   ```bash
   mkdir /data/tmp
   chown mysql:mysql /data/tmp
   chmod 775 /data/tmp
   ```

2. Modifier la conf :

   ```bash
   nano /etc/mysql/mariadb.conf.d/50-server.cnf
   ```

   Dans `[mysqld]`, ajouter :

   ```ini
   tmpdir = /data/tmp
   ```

---

## 4. Redémarrage et tests

1. Redémarrer MariaDB :

   ```bash
   systemctl start mysql
   systemctl status mysql
   ```

2. Vérifier que MariaDB utilise bien `/data` :

   ```bash
   mysql -e "SHOW VARIABLES LIKE 'datadir';"
   mysql -e "SHOW VARIABLES LIKE 'tmpdir';"
   ```

3. Vérifier l’espace disque :

   ```bash
   df -h
   ```

---

## ✅ Résultat attendu

* Le LV `/dev/mapper/data-data` est monté automatiquement sur `/data`
* `datadir` de MariaDB → `/data/mysql`
* `tmpdir` de MariaDB → `/data/tmp`
* Le disque système reste léger et MariaDB utilise bien le gros disque

---
