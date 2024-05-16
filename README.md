# Haute disponibilité

## Serveur Web + Reverse Proxy en Cluster
Schéma : 

## Mise en place
### Serveur Web
Installation des paquets nécéssaires : 
```apt update && apt install -y apache2 keepalived```
```systemctl enable apache2 keepalived```

Edition des fichiers de conf : 

Server Web A1
```nano /etc/keepalived/keepalived.conf```
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

```systemctl restart apache2 keepalived```

---
Server Web A2
```nano /etc/keepalived/keepalived.conf```
```
vrrp_instance VI_1 {
# A modifié avec le nom de votre interface
 interface eth0
 state MASTER
 virtual_router_id 51
# Priorité 100 pour celui qui répondra en premier et 100 pour l'autre
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

```systemctl restart apache2 keepalived```

Une fois cela fait l'ip virtuelle doit pouvoir être pingé et le fail over fonctionel.

### Serveur Reverse Proxy
Installation des paquets nécéssaires : 
```apt update && apt install -y haproxy keepalived```
```systemctl enable haproxy keepalived```

Modification du fichier hosts pour résoudre les noms (IP à modifié en fonction)
```nano /etc/hosts```
```
172.16.182.1 vrrpsrvweb
172.16.182.3 srvrpxa1
172.16.182.4 srvrpxa2
```

Configuration de HAProxy (A faire sur les deux serveur): 
```nano /etc/haproxy/haproxy.cfg```
```
frontend myweb
# A modifié si vous voulez un tri par ip
bind *:80
option tcplog
mode tcp
default_backend web-servers
backend web-servers
mode tcp
balance roundrobin
option tcp-check
# Changer par l'ip de votre serveur web ou votre VRRP
server vrrpsrvweb 192.168.2.13:80 check fall 3 rise 2
```

Configuration de Keepalived : 
Sur le Premier RPX : 
```nano /etc/keepalived/keepalived.conf```
```
# Define the script used to check if haproxy is still working
vrrp_script chk_haproxy {
    script "killall -0 haproxy"
    interval 2
    weight 2
}
# Configuation for the virtual interface
vrrp_instance VI_1 {
# Changer par votre interface
    interface enp0s8
# Mettre BACKUP sur les autres
    state MASTER
# Mettre 100 sur les autres
    priority 101        
    virtual_router_id 51
    smtp_alert          
    authentication {
        auth_type AH
# Mettre le meme mdp de l'autres côté
        auth_pass myPassw0rd
    }
    # The virtual ip address shared between the two loadbalancers
    virtual_ipaddress {
# Changer par la VRRP Souhaité
192.168.2.100
    }
    # Use the script above to check if we should fail over
    track_script {
        chk_haproxy
    }
}
```

```systemctl restart keepalived haproxy```
Sur le Second RPX : 
```
# Define the script used to check if haproxy is still working
vrrp_script chk_haproxy {
    script "killall -0 haproxy"
    interval 2
    weight 2
}
# Configuation for the virtual interface
vrrp_instance VI_1 {
# Changer par votre interface
    interface enp0s8
# Mettre BACKUP sur les autres
    state BACKUP
# Mettre 100 sur les autres
    priority 100      
    virtual_router_id 51
    smtp_alert          
    authentication {
        auth_type AH
# Mettre le meme mdp de l'autres côté
        auth_pass myPassw0rd
    }
    # The virtual ip address shared between the two loadbalancers
    virtual_ipaddress {
# Changer par la VRRP Souhaité
192.168.2.100
    }
    # Use the script above to check if we should fail over
    track_script {
        chk_haproxy
    }
}
```

```systemctl restart keepalived haproxy```

---
Sources : 
[Quentin Queiros](https://example.com/](https://quentinqueiros.wordpress.com/wp-content/uploads/2016/09/redondance-web.pdf ).
[learnitguide](https://www.learnitguide.net/2021/11/configure-ha-cluster-using-keepalived.html).

---
Laisse moi une ⭐️
