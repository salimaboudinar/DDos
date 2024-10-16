# Rapport Explicatif sur les Solutions de Protection contre les Attaques DDoS

## Introduction
Les attaques par déni de service distribué (DDoS) sont une menace sérieuse pour les infrastructures en ligne. Elles visent à rendre un service indisponible en inondant les serveurs de trafic malveillant. Ce rapport approfondit trois solutions principales pour se défendre contre ces attaques : l'utilisation de **fail2ban**, l'implémentation de **load balancers**, et l'utilisation de **bases de données distribuées**.

---

## 1. Utilisation de fail2ban avec un fichier log `access.log`

**Fail2ban** est un outil qui analyse les logs du serveur à la recherche d'adresses IP effectuant des tentatives de connexion suspectes. Lorsqu'une IP dépasse un certain seuil de tentatives échouées, fail2ban l'ajoute aux règles de pare-feu, bloquant ainsi l'accès à cette IP.

### a. Script Simulant l'action de fail2ban
Voici un script Python qui simule le comportement de fail2ban en surveillant le fichier `access.log` et en bloquant automatiquement les adresses IP malveillantes :

```python
import time
import re
import subprocess

LOG_FILE = 'access.log'
BAN_THRESHOLD = 5
BAN_TIME = 3600  # Temps en secondes (1 heure)
BANNED_IPS = {}

# Fonction pour ajouter une IP aux règles iptables (bloque l'IP)
def ban_ip(ip):
    subprocess.run(['sudo', 'iptables', '-A', 'INPUT', '-s', ip, '-j', 'DROP'])
    print(f"IP {ip} bannie pour comportement suspect.")

# Fonction pour surveiller le fichier log
def monitor_log():
    with open(LOG_FILE, 'r') as log_file:
        log_file.seek(0, 2)  # Aller à la fin du fichier
        while True:
            line = log_file.readline()
            if not line:
                time.sleep(1)
                continue

            # Regex pour capturer les adresses IP
            match = re.search(r'(\d{1,3}\.){3}\d{1,3}', line)
            if match:
                ip = match.group(0)
                if ip in BANNED_IPS and time.time() - BANNED_IPS[ip] > BAN_TIME:
                    del BANNED_IPS[ip]
                elif ip not in BANNED_IPS:
                    BANNED_IPS[ip] = time.time()
                    ban_ip(ip)

monitor_log()
b. Interactions de fail2ban avec les logs
Fail2ban surveille les fichiers de log générés par les serveurs web, SSH, etc. Il utilise des expressions régulières pour identifier les modèles de connexion suspectes (par exemple, plusieurs tentatives de connexion échouées en peu de temps). Lorsqu'un modèle est détecté, fail2ban déclenche une action, généralement en ajoutant l'IP malveillante aux règles du pare-feu (iptables), ce qui empêche toute connexion future.
2. Implémentation de Load Balancers et leurs algorithmes
Les load balancers répartissent le trafic entrant sur plusieurs serveurs, réduisant ainsi la charge sur un seul serveur et offrant une résilience face aux attaques DDoS.


a. Étude et mise en œuvre d'un load balancer
NGINX et HAProxy sont deux des solutions de load balancing les plus populaires. Voici un exemple de configuration basique avec NGINX :
http {
    upstream backend {
        server server1.example.com;
        server server2.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
## b. Algorithmes de répartition
- **Round Robin** : Les requêtes sont distribuées de manière égale entre les serveurs.
- **Least Connections** : Les requêtes sont envoyées au serveur ayant le moins de connexions actives.
- **IP Hashing** : Les requêtes sont distribuées en fonction de l'adresse IP du client, assurant que le même client soit dirigé vers le même serveur.

## c. Déploiement d'un load balancer
Le déploiement d'un load balancer dans un environnement distribué peut se faire en plaçant NGINX ou HAProxy en tant que serveur frontal. Lors d'une attaque DDoS, la charge du trafic est répartie, ce qui réduit la probabilité qu'un seul serveur soit submergé.

---

## 3. Bases de données distribuées et leur mode de fonctionnement
Les bases de données distribuées, telles que MongoDB et Cassandra, sont conçues pour gérer de grandes quantités de données sur plusieurs serveurs.

### a. Fonctionnement des bases de données distribuées
Ces bases de données répliquent les données sur plusieurs nœuds pour assurer la disponibilité et la résilience. Lorsqu'une requête est reçue, elle est redirigée vers le nœud approprié.

### b. Partage de la charge des requêtes
Les bases de données distribuées utilisent des techniques telles que le partitionnement et la réplication pour partager la charge des requêtes. Cela signifie qu'en cas d'attaque DDoS, la charge est répartie entre plusieurs nœuds, ce qui minimise l'impact sur un seul serveur.

### c. Avantages et limites
- **Avantages** : Haute disponibilité, résilience aux pannes, et capacité à gérer de grandes quantités de données.
- **Limites** : Complexité de mise en œuvre, latence potentielle due à la réplication des données, et le risque d'incohérences de données.

---

## Conclusion
La mise en œuvre de solutions telles que fail2ban, les load balancers et les bases de données distribuées est essentielle pour se défendre contre les attaques DDoS. Chacune de ces solutions offre des avantages uniques et, lorsqu'elles sont combinées, elles forment une stratégie robuste pour protéger les infrastructures en ligne. En comprenant et en déployant ces technologies, les organisations peuvent mieux se préparer à faire face à la menace croissante des attaques DDoS.
