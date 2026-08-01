# 🏢 CCNA2 — VLANs + Router-on-a-Stick + STP + DHCPv4 | AgroExport Bénin

![Cisco](https://img.shields.io/badge/Cisco-CCNA2-blue?style=for-the-badge&logo=cisco&logoColor=white)
![GNS3](https://img.shields.io/badge/GNS3-2.x-orange?style=for-the-badge&logo=gns3)
![Status](https://img.shields.io/badge/Status-✅%20Completed-brightgreen?style=for-the-badge)
![Protocol](https://img.shields.io/badge/Protocol-Router--on--a--Stick-purple?style=for-the-badge)
![VLANs](https://img.shields.io/badge/VLANs-2-red?style=for-the-badge)
![PCs](https://img.shields.io/badge/PCs-2-yellow?style=for-the-badge)
![DHCP](https://img.shields.io/badge/DHCPv4-2%20Pools-green?style=for-the-badge)

---

## 📋 Description

Ce TP simule l'infrastructure réseau de la société **AgroExport Bénin** 🇧🇯.
Le réseau est segmenté en **2 VLANs** représentant les services de
l'entreprise. Le routage inter-VLAN est assuré par **Router-on-a-Stick**,
la stabilité du réseau par **STP Rapid PVST+**, et la distribution des
adresses IP par **DHCPv4** avec **2 pools automatiques**. Ce TP a été
réalisé sous **GNS3** (et non Packet Tracer) avec des images Cisco réelles.

### Objectifs
- ✅ Créer et configurer **2 VLANs** (Administration, Logistique)
- ✅ Configurer **Router-on-a-Stick** sur R1 (c7200)
- ✅ Configurer **STP Rapid PVST+**
- ✅ Configurer **DHCPv4** avec 2 pools sur R1
- ✅ Tester la distribution automatique des IP sur 2 PC (VPCS)

---

## 🖥️ Équipements

| Équipement | Modèle | Nom | Rôle |
|-----------|--------|-----|------|
| 🌐 Routeur | Cisco c7200 | R1 | Router-on-a-Stick + DHCP |
| 🔌 Switch | Cisco IOSvL2 15.2 | S1-L2 | VLAN 10/20 + trunk |
| 💻 PC | VPCS | PC1 | Service Administration (VLAN10) |
| 💻 PC | VPCS | PC2 | Service Logistique (VLAN20) |

---

## 🗺️ Topologie

[R1 - c7200]
                          │
                       gi3/0
                          │
                       Gi0/0
                          │
                     [S1-L2]
                    (IOSvL2)
                     ╱          ╲
                Gi0/1          Gi0/2
                 ╱                  ╲
              PC1                  PC2
            VLAN10                VLAN20

<img width="1920" height="1080" alt="Topologie" src="https://github.com/user-attachments/assets/d32cf89f-8115-467d-a9b5-6664c28e1c75" />


---

## 🔌 Câblage

| De | Port | Vers | Port | Rôle |
|----|------|------|------|------|
| R1 | gi3/0 | S1-L2 | Gi0/0 | Trunk router-on-a-stick |
| S1-L2 | Gi0/1 | PC1 | — | Accès VLAN10 Administration |
| S1-L2 | Gi0/2 | PC2 | — | Accès VLAN20 Logistique |

---

## 📊 Plan d'adressage

| VLAN | Nom | Réseau | Passerelle | Plage DHCP |
|------|-----|--------|-----------|------------|
| 10 | ADMINISTRATION | 192.168.10.0/24 | 192.168.10.1 | .2 - .254 |
| 20 | LOGISTIQUE | 192.168.20.0/24 | 192.168.20.1 | .2 - .254 |

---

## ⚙️ Configuration complète

### 🔧 S1-L2 — Switch (VLANs + trunk)

```cisco
enable
configure terminal
hostname S1-L2

vlan 10
name ADMINISTRATION
exit
vlan 20
name LOGISTIQUE
exit

spanning-tree mode rapid-pvst

interface GigabitEthernet0/0
switchport trunk encapsulation dot1q
switchport mode trunk
exit

interface GigabitEthernet0/1
switchport mode access
switchport access vlan 10
exit

interface GigabitEthernet0/2
switchport mode access
switchport access vlan 20
exit

end
write memory
```

---

### 🔧 R1 — Router-on-a-Stick + DHCP

```cisco
enable
configure terminal
hostname R1

! Activer l'interface physique
interface GigabitEthernet3/0
no shutdown
exit

! Sous-interfaces pour chaque VLAN
interface GigabitEthernet3/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
exit

interface GigabitEthernet3/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
exit

! Pools DHCP — un par VLAN
ip dhcp excluded-address 192.168.10.1
ip dhcp pool VLAN10-ADMIN
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8
exit

ip dhcp excluded-address 192.168.20.1
ip dhcp pool VLAN20-LOGISTIQUE
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 8.8.8.8
exit

end
write memory
```

---

### 🔧 PC1 et PC2 — VPCS (DHCP)

PC1> ip dhcp
PC2> ip dhcp

---

## 🔍 Commandes de vérification

```cisco
! Vue globale du routeur
R1# show ip interface brief
R1# show ip route

! Voir les baux DHCP distribués
R1# show ip dhcp binding

! Configuration VLAN et trunk
S1-L2# show vlan brief
S1-L2# show interfaces trunk

! État du Spanning Tree
S1-L2# show spanning-tree
```

---

### 📊 Résultat attendu — show ip dhcp binding (R1)

IP address Client-ID/Hardware address Lease expiration
192.168.10.2 00e0.f778.5a01 --- Infinite ---
192.168.20.2 00e0.f778.5a02 --- Infinite ---

| Élément | Signification |
|---------|---------------|
| **192.168.10.2 attribuée à PC1** | Pool VLAN10-ADMIN fonctionnel ✅ |
| **192.168.20.2 attribuée à PC2** | Pool VLAN20-LOGISTIQUE fonctionnel ✅ |

---

## 🧪 Tests de connectivité

✅ PC1 (VLAN10) → obtient IP via DHCP (192.168.10.2)
✅ PC2 (VLAN20) → obtient IP via DHCP (192.168.20.2)

✅ PC1 → ping 192.168.20.2 (inter-VLAN via R1)


---

## 🎭 Scénario — Vérification DHCP automatique

### Sur chaque PC

PC1> release dhcp
PC1> ip dhcp


### Vérifier le bail sur le routeur

```cisco
R1# show ip dhcp binding
```

---

## 🛠️ Dépannage

| Problème | Cause | Solution |
|---------|-------|---------|
| PC n'obtient pas d'IP | gi3/0 down sur R1 | `no shutdown` sur GigabitEthernet3/0 |
| Ping inter-VLAN échoue | Sous-interface mal encapsulée | Vérifier `encapsulation dot1Q` sur chaque sous-interface |
| Trunk down entre R1 et S1-L2 | Port en mode access | `switchport mode trunk` sur Gi0/0 |
| Pool DHCP inactif | Adresse exclue mal définie | Vérifier `ip dhcp excluded-address` |

```cisco
! Diagnostic détaillé
R1# show ip dhcp pool
S1-L2# show interfaces status

! Réinitialiser un bail DHCP en conflit
R1# clear ip dhcp binding *
```

---

## 💡 Points clés à retenir

| 🔑 Commande | 📖 Rôle |
|-------------|---------|
| `interface GigabitEthernet3/0.10` | Créer une sous-interface pour le VLAN10 |
| `encapsulation dot1Q 10` | Taguer la sous-interface au VLAN10 |
| `switchport trunk encapsulation dot1q` | Activer l'encapsulation 802.1Q sur le trunk |
| `ip dhcp pool VLAN10-ADMIN` | Créer un pool DHCP par VLAN |
| `show ip dhcp binding` | Vérifier les baux distribués |
| `show spanning-tree` | Vérifier l'état des ports STP |

---

## 📊 Comparatif avant/après

| | Sans segmentation | Avec VLAN + Router-on-a-Stick |
|---|---|---|
| **Sécurité** | Tout le trafic mélangé | Services isolés (VLAN) |
| **Adressage** | Manuel | Automatique (DHCP, 2 pools) |
| **Routage inter-VLAN** | ❌ Impossible | ✅ Via R1 |
| **Stabilité réseau** | ❌ Boucles possibles | ✅ Rapid PVST+ actif |

---

## 🛠️ Outils

![GNS3](https://img.shields.io/badge/GNS3-2.x-orange?style=flat-square&logo=gns3)
![Cisco IOS](https://img.shields.io/badge/Cisco%20IOS-15.x-blue?style=flat-square)
![GitHub](https://img.shields.io/badge/GitHub-black?style=flat-square&logo=github)

---

## 👨‍💻 Auteur

**Urbain Sedami Landjidé**
🎓 Étudiant en 2ème année — Licence Professionnelle
📡 Réseaux Informatique Mobilité Sécurité (RMS)
🏫 Cisco Networking Academy
📍 Cotonou, Bénin 🇧🇯

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connecter-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/urbain-sedami-landjide-9b49043a8/)

---

## 📄 Licence

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Libre d'utilisation pour l'apprentissage et la formation réseau.
