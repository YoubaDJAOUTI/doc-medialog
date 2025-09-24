

# 📘 Documentation : Configuration LVM et migration MariaDB vers `/data`

## 1. Préparation du disque et création du LVM

1. Vérifier les disques présents :

   ```bash
   lsblk
   df -h
   ```

2. Créer une partition LVM sur le disque `/dev/vdb` :

   ```bash
   /sbin/fdisk /dev/vdb
   ```

   * `n` → nouvelle partition primaire
   * `p` → partition primaire
   * Enter pour taille complète
   * `t` → changer le type en `8e` (Linux LVM)
   * `w` → écrire et quitter

3. Initialiser la partition comme **Physical Volume (PV)** :

   ```bash
   pvcreate /dev/vdb1
   ```

4. Créer un **Volume Group (VG)** appelé `data` :

   ```bash
   vgcreate data /dev/vdb1
   ```

5. Créer un **Logical Volume (LV)** prenant tout l’espace :

   ```bash
   lvcreate -n data -l +100%FREE data
   ```

6. Vérifier la création :

   ```bash
   lvs
   ```

7. Formater le LV en ext4 :

   ```bash
   mkfs.ext4 /dev/data/data
   ```

8. Monter le LV sur `/data` :

   ```bash
   mkdir /data
   mount -a
   ```

9. Ajouter au fichier `/etc/fstab` pour un montage automatique :

   ```fstab
   /dev/data/data   /data   ext4   defaults   0   2
   ```

---

## 2. Migration de MariaDB vers `/data`

1. Arrêter le service :

   ```bash
   service mysql stop
   ```

2. Déplacer le répertoire MySQL :

   ```bash
   cd /var/lib/
   mv mysql /data/
   ln -s /data/mysql mysql
   ```

3. Vérifier :

   ```bash
   ls -l /var/lib/
   ```

4. Redémarrer le service :

   ```bash
   service mysql start
   ```

---

## 3. Configuration d’un répertoire temporaire dédié

1. Créer un dossier temporaire :

   ```bash
   cd /data
   mkdir tmp
   chown mysql:mysql tmp
   chmod 775 tmp
   ```

2. Modifier la configuration MariaDB (`/etc/mysql/mariadb.conf.d/50-server.cnf`) :
   Dans la section `[mysqld]`, ajouter :

   ```ini
   tmpdir = /data/tmp
   ```

3. Redémarrer MariaDB :

   ```bash
   service mysql restart
   ```

---

## 4. Vérifications

1. Vérifier l’espace disque :

   ```bash
   df -h
   ```

2. Vérifier que MySQL utilise le bon `datadir` :

   ```bash
   mysql -e "SHOW VARIABLES LIKE 'datadir';"
   ```

3. Vérifier que le `tmpdir` est bien `/data/tmp` :

   ```bash
   mysql -e "SHOW VARIABLES LIKE 'tmpdir';"
   ```

---

## ✅ Résultat attendu

* `/data` est monté sur le disque LVM de 400 Go
* `datadir` de MariaDB est déplacé vers `/data/mysql`
* Les fichiers temporaires de MariaDB sont écrits dans `/data/tmp`
* Le disque système (32 Go) reste disponible pour l’OS uniquement

---
