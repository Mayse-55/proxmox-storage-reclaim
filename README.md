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

## 📊 Exemple concret de récupération d'espace

### Situation de départ
```
Disque total : 256 Go
├── local (root)    : 83 Go   → ISOs, templates, backups
└── local-lvm (data): 173 Go  → Disques VMs et containers
```

### ⚠️ Le problème

Avec cette configuration par défaut :
- **173 Go** sont alloués pour les VMs et containers dans `local-lvm`
- **Seulement 83 Go** pour les backups, ISOs et templates dans `local`

**Conséquence** : Si vous avez des VMs qui utilisent 100 Go, vous ne pourrez stocker qu'un seul backup complet (voire aucun si vous avez plusieurs VMs) ! L'espace pour les backups est **largement insuffisant** par rapport à l'espace disponible pour les VMs.

### Après la manipulation
```
Disque total : 256 Go
└── local (root)    : 248 Go  → Tout (VMs, containers, backups, ISOs, templates)
    (8 Go réservés pour le système)
```

**Avantages** :
- ✅ **248 Go flexibles** : l'espace s'adapte automatiquement à vos besoins
- ✅ **Backups possibles** : assez d'espace pour sauvegarder vos VMs
- ✅ **Plus simple** : un seul espace de stockage à gérer

### 📈 Comparaison

| Configuration | Backups/ISOs | VMs/Containers | Total utilisable |
|---------------|--------------|----------------|------------------|
| **Avant** ⚠️ | 83 Go (32%) | 173 Go (68%) | 256 Go cloisonné |
| **Après** ✅ | ← 248 Go flexibles → | | 248 Go unifié |

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

---

## 📦 Étape importante : Configurer le stockage `local`

⚠️ **N'oubliez pas** de mettre à jour la configuration du stockage `local` pour qu'il puisse gérer tous les types de contenu qui étaient auparavant dans `local-lvm`.

### 🖱️ Configuration via l'interface web

Vous pouvez aussi le faire via l'interface Proxmox :
1. Allez dans **Datacenter** → **Storage**
2. Sélectionnez le stockage **local**
3. Cliquez sur **Edit**
4. Dans **Content**, cochez tous les types qui étaient disponibles dans `local-lvm` :
   - ✅ Disk image
   - ✅ Container
   - ✅ VZDump backup file
   - ✅ ISO image
   - ✅ Container template

💡 **Rappel** : Le contenu typique de `local-lvm` incluait généralement `rootdir` (containers) et `images` (disques de VMs). Assurez-vous de les ajouter au stockage `local` pour pouvoir créer des VMs et containers sur ce stockage.
