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

## 🚀 Récupération espace disque

### Étape 1 : Récupération de l'espace
```bash
lvextend -l +100%FREE /dev/pve/root && resize2fs /dev/pve/root
```

### Étape 2 : Supprimer Local-lvm

✅ 1. Ouvre le fichier de configuration :
```bash
nano /etc/pve/storage.cfg
```
✅ 2. Repère et supprime le bloc qui ressemble à :
```bash
lvmthin: local-lvm
    thinpool data
    vgname pve
    content rootdir,images
```

Tu peux aussi le commenter avec # au début de chaque ligne si tu préfères temporairement le désactiver.

>[!caution]
>⚠️ Attention à ne pas supprimer d'autres blocs comme celui de local, qui est souvent le stockage basé sur /var/lib/vz.
