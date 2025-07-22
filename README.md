# proxmox-disk-merge
**1. Supprimer tous les volumes data :**
```bash
lvremove /dev/pve/data -y && lvremove /dev/pve/data_tdata -y && lvremove /dev/pve/data_tmeta -y
```

**2. Récupérer tout l'espace libre :**
```bash
lvextend -l +100%FREE /dev/pve/root && resize2fs /dev/pve/root
```

Exécutez d'abord la première pour nettoyer tous les volumes data, puis la deuxième pour donner tout l'espace libéré au volume root.

Si la première commande donne des erreurs parce que certains volumes n'existent plus, c'est normal, continuez avec la deuxième commande.
