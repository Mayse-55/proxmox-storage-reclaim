# 🚀 Proxmox Disk Merge

> **Script optimisé pour récupérer 100% de l'espace disque dans Proxmox VE**

[![Proxmox](https://img.shields.io/badge/Proxmox-VE-orange?style=flat-square&logo=proxmox)](https://www.proxmox.com/)
[![LVM](https://img.shields.io/badge/Storage-LVM-blue?style=flat-square)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux))
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

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

## 🚀 Installation rapide

### Étape 1 : Nettoyage des volumes LVM
```bash
lvremove /dev/pve/data -y && lvremove /dev/pve/data_tdata -y && lvremove /dev/pve/data_tmeta -y
```

### Étape 2 : Récupération de l'espace
```bash
lvextend -l +100%FREE /dev/pve/root && resize2fs /dev/pve/root
```

---

## 📝 Instructions détaillées

### 🔍 Pré-requis
- Proxmox VE installé avec configuration LVM par défaut
- Accès SSH root au serveur
- **Sauvegarde des VMs critiques** (recommandé)

### ⚡ Exécution
1. **Connectez-vous** en SSH à votre serveur Proxmox
2. **Exécutez la première commande** pour supprimer les volumes inutiles
3. **Exécutez la seconde commande** pour récupérer l'espace libéré
4. **Vérifiez le résultat** avec `df -h /`

### 🛠️ Diagnostic (optionnel)
```bash
# Vérifier l'état actuel
lsblk && df -h && vgs && lvs

# Vérifier après l'opération
df -h / && vgs
```

---

## ⚠️ Avertissements importants

| ⚠️ | **ATTENTION** |
|---|---|
| 🔥 | Cette opération supprime définitivement les volumes `local-lvm` |
| 💾 | Sauvegardez vos VMs importantes avant d'exécuter |
| 🚫 | Les VMs stockées sur `local-lvm` seront perdues |
| ✅ | Les VMs sur stockage `local` ne sont PAS affectées |

---

## 🐛 Résolution des problèmes

### Erreur "Failed to find logical volume"
```bash
# Normal si le volume n'existe plus, continuez avec l'étape 2
```

### Commande qui ne se termine pas
```bash
# Interrompre avec Ctrl+C puis relancer :
e2fsck -f /dev/pve/root && resize2fs /dev/pve/root
```

### Vérification des VMs avant suppression
```bash
# Lister les VMs existantes
qm list

# Vérifier le stockage utilisé
pvesm list local-lvm 2>/dev/null || echo "Pas de VMs sur local-lvm"
```

---

## 🎯 Cas d'usage typiques

- ✅ Installation Proxmox par défaut avec `local` + `local-lvm`
- ✅ Espace disque fragmenté non utilisé
- ✅ Besoin de simplifier la gestion du stockage
- ✅ Optimisation pour serveurs homelab

---

## 📈 Contributions

Les contributions sont les bienvenues ! N'hésitez pas à :
- 🐛 Signaler des bugs
- 💡 Proposer des améliorations
- 📖 Améliorer la documentation
- ⭐ Mettre une étoile si ce script vous a été utile

---

## 📄 License

Ce projet est sous licence MIT. Voir le fichier [LICENSE](LICENSE) pour plus de détails.

---

<div align="center">

**Made with ❤️ for the Proxmox community**

[⭐ Star ce repo](../../stargazers) • [🐛 Signaler un bug](../../issues) • [💬 Discussions](../../discussions)

</div>
