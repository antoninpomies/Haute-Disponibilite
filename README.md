# Haute disponibilité

## Serveur Web + Reverse Proxy en Cluster
Schéma : 

## Mise en place
### Serveur Web
Installation des paquets nécéssaires : 
```apt update && apt install -y apache2 keepalived```

Edition des fichiers de conf : 
```
vrrp_instance VI_1 {
# A modifié avec le nom de votre interface
 interface eth0
 state MASTER
 virtual_router_id 51
# Priorité 101 pour celui qui répondra en premier et 100 pour l'autre
 priority 101
 authentication {
 auth_type AH
# Spécifié le mot de passe pour l'authentification (Mot de passe a définir et a mettre sur les deux
 auth_pass "pass"
 }
 virtual_ipaddress {
# IP virtuelle a spécifié
 172.16.4.108
 }
```
