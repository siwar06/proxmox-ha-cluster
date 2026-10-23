# Proxmox VE : infrastructure virtualisée, cluster, HA et automatisation

Projet d'intégration réalisé à l'**ISET Charguia** (Licence RSI, année universitaire 2025-2026).
Mise en place d'une infrastructure de virtualisation complète basée sur **Proxmox VE 9.1** : VMs, conteneurs, cluster multi-nœuds, migration, sauvegarde, haute disponibilité avec Ceph, et automatisation avec Ansible, Vagrant et Terraform.

## Architecture

![Architecture globale](docs/screenshots/54-architecture-globale.png)

![Outils d'automatisation](docs/screenshots/53-automation-tools-schema.png)

## Technologies

Proxmox VE 9.1 · KVM · LXC · Corosync · Ceph · Ansible · Vagrant · Terraform · VirtualBox · Linux (Debian)

## Étapes réalisées

### 1. Installation et prise en main
- Installation de Proxmox VE sur VirtualBox avec virtualisation imbriquée (nested VT-x)
- Création d'une VM KVM et d'un conteneur LXC (Alpine)
- Création et clonage de templates (VM et conteneur)

| | | |
|---|---|---|
| ![](docs/screenshots/03-nested-virtualization.png) | ![](docs/screenshots/05-web-ui.png) | ![](docs/screenshots/08-create-vm-confirm.png) |
| Nested virtualization | Interface web | Création d'une VM |

![](docs/screenshots/13-create-lxc-confirm.png)

### 2. Cluster et migration
- Cluster `my-cluster` (Corosync), 2 nœuds `pve` et `pve2`
- VM Tiny Core Linux (ID 105) migrée **à froid** puis **à chaud**

![](docs/screenshots/20-two-nodes-active.png)
![](docs/screenshots/26-cold-migration.png)
![](docs/screenshots/28-live-migration.png)

| Critère | Migration à froid | Migration à chaud |
|---|---|---|
| État de la VM | Éteinte | Allumée |
| Interruption | Oui | Non |
| Transfert | Disque complet | RAM synchronisée |

### 3. Snapshots, backups et haute disponibilité
- Snapshots avec RAM, sauvegardes (mode snapshot, compression ZSTD)
- Cluster à 3 nœuds pour le quorum, stockage partagé **Ceph** (OSD sur chaque nœud, pool répliqué ×3)
- VM 105 protégée par HA avec basculement automatique

![](docs/screenshots/30-snapshots-tab.png)
![](docs/screenshots/33-backup-config.png)
![](docs/screenshots/34-quorum-3-nodes.png)
![](docs/screenshots/36-ceph-health-ok.png)
![](docs/screenshots/37-ceph-osd.png)

**Test de basculement (nœud défaillant)** :

![](docs/screenshots/39-failover-in-progress.png)
![](docs/screenshots/40-failover-success.png)

### 4. Automatisation
| Outil | Rôle | Réalisation |
|---|---|---|
| Ansible | Gestion de configuration (SSH, agentless) | Playbook d'installation de `htop` sur pve2 |
| Vagrant | Création de VMs depuis un Vagrantfile | Vagrantfile validé (Debian Bookworm + nginx) |
| Terraform | Infrastructure as Code via l'API Proxmox | VM créée avec `terraform apply` |

![](docs/screenshots/42-ansible-ping.png)
![](docs/screenshots/44-ansible-playbook-run.png)
![](docs/screenshots/47-vagrant-validate.png)
![](docs/screenshots/51-terraform-plan.png)
![](docs/screenshots/52-terraform-apply.png)

## Problèmes rencontrés et solutions

| # | Problème | Solution |
|---|---|---|
| 1 | Service `networking` inactif après modification de `/etc/network/interfaces` | Redémarrage du service et vérification avec `systemctl status` |
| 2 | Échec du join au cluster (`no quorum`) | `pvecm expected 1` pour forcer le quorum, puis relance du join |
| 3 | Hostname incohérent dans `/etc/hosts` après renommage du nœud | Mise à jour manuelle de la ligne du nœud |
| 4 | IP réelle de `vmbr0` différente de celle utilisée pour accéder à l'interface | Correction de `/etc/network/interfaces` |
| 5 | KVM indisponible en virtualisation imbriquée | `qm set <id> --kvm 0 --cpu qemu64` (émulation QEMU) |
| 6 | Terraform : format `qcow2` incompatible avec LVM-Thin | `file_format = "raw"` dans le bloc `disk` |
| 7 | Terraform : accélération KVM non disponible | Type de CPU `qemu64` dans le bloc `cpu` |

Captures associées : [`docs/screenshots/`](docs/screenshots) (préfixe `6x` et `7x`).

## Compétences acquises

Virtualisation (KVM/LXC) · Clustering et quorum · Migration de VMs · Stockage distribué Ceph · Haute disponibilité · Sauvegarde et restauration · Infrastructure as Code · Dépannage système et réseau

## Contenu du dépôt

```
.
├── README.md
├── .gitignore
└── docs/
    └── screenshots/   # captures du projet
```
