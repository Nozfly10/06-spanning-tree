# 06 — Redondance Réseau avec Spanning Tree Protocol (STP)

## 🎯 Objectif
Mettre en place le protocole STP/RSTP sur une infrastructure Cisco afin d'assurer la redondance réseau tout en évitant les boucles de commutation.

---

## 🏗️ Architecture

```
                    [Switch Root - SW1]
                   /                   \
            (lien principal)      (lien bloqué STP)
                 /                         \
         [Switch SW2]  <--- lien bloqué --- [Switch SW3]
              |                                  |
        [VLAN 10/20]                       [VLAN 30]
```

---

## 📋 Rôles STP

| Switch | Rôle | Bridge Priority |
|---|---|---|
| SW1 | Root Bridge | 4096 (le plus bas) |
| SW2 | Designated | 32768 (défaut) |
| SW3 | Non-designated | 32768 (défaut) |

---

## ⚙️ Configuration STP / RSTP

### Définir le Root Bridge
```
! SW1 — Root Bridge principal
SW1(config)# spanning-tree vlan 10 priority 4096
SW1(config)# spanning-tree vlan 20 priority 4096
SW1(config)# spanning-tree vlan 30 priority 4096

! SW2 — Root Bridge secondaire (backup)
SW2(config)# spanning-tree vlan 10 priority 8192
SW2(config)# spanning-tree vlan 20 priority 8192
```

### Activer RSTP (plus rapide que STP classique)
```
SW1(config)# spanning-tree mode rapid-pvst
SW2(config)# spanning-tree mode rapid-pvst
SW3(config)# spanning-tree mode rapid-pvst
```

### PortFast (ports vers postes clients)
```
! Accélère la convergence sur les ports end-devices
SW2(config)# interface range fa0/1-20
SW2(config-if-range)# spanning-tree portfast
SW2(config-if-range)# spanning-tree bpduguard enable
```

### BPDU Guard (sécurité)
```
! Désactive le port si un switch non autorisé est branché
SW2(config)# spanning-tree portfast bpduguard default
```

### Optimisation — Coût des liens
```
! Favoriser un lien Gigabit plutôt que FastEthernet
SW2(config)# interface gigabitEthernet 0/1
SW2(config-if)# spanning-tree vlan 10 cost 4
```

---

## ✅ Vérification

```
! Vue globale STP
SW1# show spanning-tree

! Par VLAN
SW1# show spanning-tree vlan 10

! Root Bridge actuel
SW1# show spanning-tree | include Root

! Ports et leurs rôles
SW2# show spanning-tree detail

! Vérifier BPDU Guard
SW2# show spanning-tree inconsistentports
```

---

## 🔁 Test de bascule (failover)

```bash
# 1. Débrancher le lien principal du Root Bridge
# 2. Observer la reconvergence RSTP (< 2 secondes)
# 3. Vérifier le nouveau Root Bridge
SW2# show spanning-tree | include Root

# 4. Rebrancher le lien et vérifier le retour
```

---

## 🎓 Compétences acquises

- Compréhension du protocole STP et RSTP
- Élection et configuration du Root Bridge
- Gestion des ports (Root, Designated, Blocked)
- Sécurisation avec BPDU Guard et PortFast
- Test de redondance et bascule automatique
