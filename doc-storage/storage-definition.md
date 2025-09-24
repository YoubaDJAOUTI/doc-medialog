
## 🔹 Paramètres de la section **Disks**

### 1. **Bus/Device (VirtIO Block, SCSI, SATA, IDE)**

* C’est la façon dont le disque est “branché” à la VM.
* Les choix possibles :

  * **IDE** : ancien standard, très compatible mais lent (à éviter sauf cas particuliers).
  * **SATA** : comme un disque dur moderne branché en SATA, bonne compatibilité, mais un peu plus lent que VirtIO.
  * **SCSI** : performant, permet aussi des fonctionnalités avancées (snapshots, hot-plug).
    → associé au **SCSI Controller** choisi dans l’onglet Système.
  * **VirtIO Block** : interface para-virtualisée, très rapide, mais nécessite que les drivers soient présents dans l’OS (dans Debian ils le sont par défaut).
* 👉 Pour Debian 13 : **VirtIO SCSI** ou **VirtIO Block** sont recommandés.

---

### 2. **Storage**

* L’emplacement où sera stocké le disque virtuel.
* Dépend de ta config Proxmox :

  * **local** : stockage sur le disque du serveur (généralement dans `/var/lib/vz`).
  * **local-lvm** : stockage sur un volume LVM (meilleur pour performance et snapshots).
  * **ZFS pool** : si tu as configuré ZFS.
  * **NFS/iSCSI/Ceph** : si tu utilises un stockage réseau partagé.
* 👉 Choisir celui qui correspond à ton besoin (performance vs capacité).

---

### 3. **Disk Size**

* Taille du disque virtuel.
* Exemple : 20G, 50G…
* Tu peux **agrandir plus tard**, mais **pas réduire** (sauf opérations complexes).

---

### 4. **Format**

* Détermine comment le fichier disque est stocké sur Proxmox :

  * **raw** : format brut → rapide, efficace, mais pas de snapshots si stockage classique.
  * **qcow2** : format avancé → supporte snapshots, compression, allocation dynamique.
    (plus lent que raw sur certains workloads).
* 👉 Si tu veux des snapshots → **qcow2**.
  Si tu veux performance max → **raw** (surtout si LVM/ZFS derrière).

---

### 5. **Cache**

* Définit comment le cache disque de la VM interagit avec l’hôte :

  * **Default (no cache)** : recommandé, sécurité et performance équilibrées.
  * **Writeback** : rapide, mais risque de corruption si coupure brutale (il faut avoir un bon UPS).
  * **Writethrough** : plus sûr mais plus lent (chaque écriture passe directement sur le disque hôte).
* 👉 Généralement, laisse **Default** sauf besoin particulier.

---

### 6. **Discard / SSD Emulation**

* **Discard (TRIM/UNMAP)** : permet de libérer l’espace disque inutilisé par la VM sur le stockage.
  Utile si stockage finement provisionné (thin provisioning).
* **SSD emulation** : fait croire à la VM que son disque est un SSD → Debian active des optimisations spécifiques.
* 👉 Pour Debian 13 sur disque moderne → active **SSD emulation** et **Discard** si ton backend (ZFS, LVM-thin, Ceph, etc.) le supporte.

---

## ✅ Recommandation Debian 13 (disque système)

* **Bus/Device** : VirtIO SCSI
* **Storage** : local-lvm (si tu veux snapshots) ou ZFS si dispo
* **Disk Size** : min 20G (plus selon besoin)
* **Format** : qcow2 (souplesse + snapshots)
* **Cache** : Default
* **Discard + SSD emulation** : activés

---

---

## 🔹 Qu’est-ce que **IO Thread** ?

* Chaque disque virtuel de ta VM doit traiter des **opérations d’entrée/sortie (I/O)** (lectures/écritures).
* Par défaut, ces I/O passent par un **thread unique** qui gère tous les disques attachés à la VM.
* Quand tu coches **IO Thread**, Proxmox va créer **un thread séparé par disque**.

---

## 🔹 Pourquoi c’est utile ?

* Cela permet de **paralléliser les accès disques** :

  * Si tu as plusieurs disques virtuels, ils peuvent travailler indépendamment sans se bloquer mutuellement.
  * Sur des workloads intensifs (ex. bases de données, gros traitements I/O), ça améliore les performances.
* Sur les CPU modernes multi-cœurs, ça réduit la contention.

---

## 🔹 Quand l’activer ?

* **Recommandé** si :

  * Tu utilises **VirtIO SCSI** (c’est optimisé pour ça).
  * Tu as plusieurs disques dans la VM.
  * Ta VM fait beaucoup d’accès disque simultanés.
* **Pas utile** (voire inutilement consommateur) si :

  * Tu n’as qu’un seul petit disque système avec peu de charge.
  * Tu fais tourner juste un petit Debian serveur avec des services légers.

---

## 🔹 Exemple concret

* VM Debian 13 avec **un seul disque système 20 Go** pour SSH, Nginx, MariaDB léger → **IO Thread pas nécessaire**.
* VM avec **plusieurs disques** (un pour OS, un pour base de données, un pour logs) → **IO Thread recommandé**.
* VM base de données PostgreSQL/MySQL avec fort trafic disque → **toujours activer IO Thread**.

---

👉 Résumé simple :

* **IO Thread OFF** → simple, moins de threads, suffisant pour petites VM.
* **IO Thread ON** → meilleures perfs quand plusieurs disques ou gros I/O, recommandé pour VirtIO SCSI.

---

