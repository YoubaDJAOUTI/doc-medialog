
### 📌 Infos générales (distribution + version)

```bash
lsb_release -a
```

ou

```bash
cat /etc/os-release
```

---

### 📌 Kernel et architecture

```bash
uname -a
```

ou plus lisible :

```bash
uname -r   # version du noyau
uname -m   # architecture (x86_64, arm64, etc.)
```

---

### 📌 Matériel

* Processeur :

```bash
lscpu
```

* Mémoire RAM :

```bash
free -h
```

* Disques :

```bash
lsblk
```

---

### 📌 Tout en un (résumé complet)

Si tu veux un résumé pratique :

```bash
neofetch
```

*(il faut l’installer avec `sudo apt install neofetch`)*
