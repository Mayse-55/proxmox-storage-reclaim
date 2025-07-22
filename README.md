# 🚀 Proxmox Disk Merge
[![Proxmox](https://img.shields.io/badge/Proxmox-VE-orange?style=flat-square&logo=proxmox)](https://www.proxmox.com/)
[![LVM](https://img.shields.io/badge/Storage-LVM-blue?style=flat-square)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux))

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

### Étape 1 : Nettoyage des volumes LVM
```bash
lvremove /dev/pve/data -y && lvremove /dev/pve/data_tdata -y && lvremove /dev/pve/data_tmeta -y
```
> Si la première commande donne des erreurs parce que certains volumes n'existent plus, c'est normal, continuez avec la deuxième commande.

### Étape 2 : Récupération de l'espace
```bash
lvextend -l +100%FREE /dev/pve/root && resize2fs /dev/pve/root
```
