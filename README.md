# 🚀 Proxmox Disk Merge
[![Proxmox](https://img.shields.io/badge/Proxmox-VE-orange?style=flat-square&logo=proxmox)](https://www.proxmox.com/)
[![LVM](https://img.shields.io/badge/Storage-LVM-blue?style=flat-square)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux))

# 🧾 Informations

* 📦 Proxmox : 9.0.0
* 🐧 Distribution : Debian 13 trixie

> [!caution]
> Ces commandes ne sont pas sans risque et **peuvent entraîner une perte de données**.  
> Assurez-vous d’avoir sauvegardé vos fichiers importants avant de continuer.

## 📋 Description

Ce script permet de **fusionner et optimiser l'espace disque** dans Proxmox VE en supprimant les volumes LVM fragmentés et en consolidant tout l'espace disponible sur le volume principal.

### ✨ Avantages
- 🎯 **Récupération complète** de l'espace disque non utilisé
- 🧹 **Nettoyage automatique** des volumes LVM obsolètes  
- ⚡ **Consolidation** sur un seul volume unifié
- 🔧 **Simple** : seulement 2 commandes à exécuter

### 📊 Exemple concret
```
Avant : 83 Go utilisables / 256 Go total (67% d'espace perdu)
Après  : 248 Go utilisables / 256 Go total (97% d'espace récupéré)
```
---

# 🚀 Récupération de l'espace disque dans Proxmox

## 📋 Étapes pour Proxmox 8

### 🧩 Étape 1 : Supprimer le thin pool `data`
Supprimez le volume logique `data` pour libérer l'espace :
```bash
lvremove /dev/pve/data
```
⚠️ **Attention** : Cette commande supprime définitivement le thin pool. Assurez-vous qu'aucune VM ou container ne l'utilise.

### 🔧 Étape 2 : Étendre le volume `root` avec l'espace libéré
Allouez **tout l'espace libre** du volume group (`pve`) à la partition `root`, puis redimensionnez le système de fichiers :
```bash
lvextend -l +100%FREE /dev/pve/root && resize2fs /dev/pve/root
```
💡 **Note** : Cette commande suppose que vous utilisez un système de fichiers **ext4**. Si vous utilisez **xfs**, remplacez `resize2fs` par `xfs_growfs /dev/pve/root`.

### 🧹 Étape 3 : Supprimer `local-lvm` de la configuration Proxmox
1. Éditez le fichier de configuration des stockages :
   ```bash
   nano /etc/pve/storage.cfg
   ```
2. Recherchez et supprimez le bloc suivant ou désactivez-le avec `#` :
   ```ini
   lvmthin: local-lvm
       thinpool data
       vgname pve
       content rootdir,images
   ```

❌ **Ne supprimez pas d'autres sections comme `local`, qui correspond souvent au stockage principal.**

---

## 📋 Étapes pour Proxmox 9

### 🧩 Étape 1 : Étendre le volume `root` avec l'espace libéré
Allouez **tout l'espace libre** du volume group (`pve`) à la partition `root`, puis redimensionnez le système de fichiers :
```bash
lvextend -l +100%FREE /dev/pve/root && resize2fs /dev/pve/root
```
💡 **Note** : Cette commande suppose que vous utilisez un système de fichiers **ext4**. Si vous utilisez **xfs**, remplacez `resize2fs` par `xfs_growfs /dev/pve/root`.

### 🧹 Étape 2 : Supprimer `local-lvm` de la configuration Proxmox
1. Éditez le fichier de configuration des stockages :
   ```bash
   nano /etc/pve/storage.cfg
   ```
2. Recherchez et supprimez le bloc suivant ou désactivez-le avec `#` :
   ```ini
   lvmthin: local-lvm
       thinpool data
       vgname pve
       content rootdir,images
   ```

❌ **Ne supprimez pas d'autres sections comme `local`, qui correspond souvent au stockage principal.**
