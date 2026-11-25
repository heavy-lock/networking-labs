# HEAVYLOCK — Laboratoire réseau 01 : Comprendre le modèle TCP/IP

🇬🇧 [Read the English version](README.md)

> [!NOTE]
> **Niveau :** débutant · **Durée estimée :** 1 h 30 à 2 h · **Environnement :** Linux (Debian/Ubuntu, ou RHEL/Rocky/AlmaLinux avec `dnf`)

## 🎯 Objectif du laboratoire

Ce laboratoire t'apprend à comprendre comment les machines communiquent sur un réseau grâce au modèle TCP/IP.

Tu vas :

- explorer les quatre couches du modèle ;
- observer les informations réseau de ta propre machine ;
- analyser de vraies communications TCP et UDP ;
- diagnostiquer deux pannes réseau simples.

> [!IMPORTANT]
> **Le but n'est pas de tout lire ligne par ligne.** Les commandes réseau affichent beaucoup d'informations, et la plupart ne te servent pas encore. À chaque exercice, une ligne **👁️ Regarde seulement** t'indique les 2 ou 3 éléments qui comptent. Le reste, tu peux l'ignorer pour l'instant : tu y reviendras quand tu en auras besoin.

> [!NOTE]
> Les résultats affichés dans ce laboratoire sont des **exemples**. Tes adresses IP, tes interfaces, tes ports et certaines réponses seront différents. Ce n'est pas une erreur : l'objectif est de comprendre l'idée, pas de reproduire exactement les mêmes résultats.

> [!IMPORTANT]
> **Tu n'as pas besoin de mémoriser les commandes.** Tu dois comprendre à quoi elles servent, savoir lire leur documentation (`man`) et repérer l'essentiel dans leurs résultats.

---

## ⭐ L'essentiel de ce laboratoire

Si tu ne devais retenir que cinq idées :

1. Le modèle TCP/IP a **quatre couches** : Application, Transport, Internet, Accès réseau.
2. Une **adresse IP** identifie une machine sur un réseau ; une **adresse MAC** identifie une interface sur le réseau local.
3. La **table de routage** décide par où partent les paquets.
4. **TCP** ouvre une connexion avant d'envoyer des données (**three-way handshake**) ; **UDP** envoie directement, sans connexion.
5. Quand une communication échoue, la première question est : **à quelle couche se situe le problème ?**

---

## 🧭 Plan du laboratoire

| Partie | Thème | Ce que tu fais |
|---|---|---|
| **Niveau 1** | Les quatre couches TCP/IP | Tu observes ta machine, couche par couche. |
| **Niveau 2** | Une communication TCP réelle | Tu crées un serveur HTTP local et tu captures le handshake. |
| **Niveau 3** | Une communication UDP | Tu envoies un datagramme et tu le compares à TCP. |
| **Niveau 4** | Diagnostiquer une panne | Tu provoques deux pannes volontaires et tu les interprètes. |

## 🧰 Avant de commencer

**Ce dont tu as besoin :**

- une machine Linux (physique, virtuelle ou VPS) avec un accès `sudo` ;
- un terminal (pour certains exercices, tu en ouvriras trois en même temps) ;
- les paquets `python3`, `curl`, `tcpdump` et `nc` (netcat).

**Installation si un outil manque :**

| Système | Commande |
|---|---|
| Debian / Ubuntu | `sudo apt install python3 curl tcpdump netcat-openbsd` |
| RHEL / Rocky / AlmaLinux | `sudo dnf install python3 curl tcpdump nmap-ncat` |

> [!WARNING]
> Sous RHEL, la commande `nc` est fournie par le paquet `nmap-ncat`. Sa syntaxe est proche de celle de `netcat-openbsd`, mais pas toujours identique. Si une option ne fonctionne pas, consulte `man nc`.

**Légende :**

| Symbole | Signification |
|---|---|
| 🧪 | La commande à exécuter |
| 👀 | Le résultat que tu peux obtenir |
| 👁️ | Ce que tu dois regarder dans le résultat (le reste peut être ignoré) |
| 🧠 | Comment comprendre ce résultat |
| ✅ | L'idée à retenir |
| 🔎 | Pour aller plus loin (optionnel) |

---

# 🧱 Niveau 1 — Explorer les quatre couches TCP/IP

| Ordre | Couche | Rôle principal |
|---|---|---|
| 4 | **Application** | Fournit les services réseau aux applications. |
| 3 | **Transport** | Assure la communication entre les applications avec TCP ou UDP. |
| 2 | **Internet** | Gère l'adressage IP et le routage des paquets. |
| 1 | **Accès réseau** | Permet la transmission des données sur le réseau local. |

Quand une machine **envoie** des données, celles-ci descendent les couches, de l'Application vers l'Accès réseau. À la **réception**, elles remontent dans l'ordre inverse.

---

## 1️⃣ Couche Application

🎯 **Objectif :** observer comment un nom de domaine est résolu et comment un serveur web répond à une requête HTTP.

### Exercice 1 — Résoudre un nom de domaine

🧪 **Commande :**

```bash
getent hosts example.com
```

👀 **Résultat possible :**

```text
93.184.216.34 example.com
```

Tu peux aussi obtenir une adresse IPv6 (par exemple `2606:4700:10::6814:179a example.com`), plusieurs adresses, ou **aucun résultat** si la résolution échoue.

👁️ **Regarde seulement :** l'adresse IP, puis le nom à droite.

🧠 **Comment comprendre le résultat ?**

- Une adresse IPv4 a quatre nombres séparés par des points.
- Une adresse IPv6 est écrite en hexadécimal, séparée par des deux-points.
- Un nom de domaine peut avoir des enregistrements `A` (IPv4) et `AAAA` (IPv6). Selon ta connectivité IPv6, ton résolveur DNS et ta configuration, tu peux obtenir l'un ou l'autre.

> [!TIP]
> `getent hosts` peut ne renvoyer qu'une seule famille d'adresses. Essaie aussi `getent ahosts example.com` pour comparer. Vérifie le comportement exact avec `man getent`.

✅ **À retenir :** un nom de domaine peut être associé à une ou plusieurs adresses IP, en IPv4 et/ou en IPv6.

### Exercice 2 — Envoyer une requête HTTP

🧪 **Commande :**

```bash
curl -I https://example.com
```

L'option `-I` demande uniquement les en-têtes de la réponse.

👀 **Résultat possible :**

```text
HTTP/2 200
content-type: text/html; charset=utf-8
server: cloudflare
```

👁️ **Regarde seulement :** la première ligne, et surtout le code (`200`).

🧠 **Comment comprendre le résultat ?**

| Code ou message | Signification |
|---|---|
| `200` | La requête a réussi. |
| `301` ou `302` | Le serveur indique une redirection. |
| `403` | L'accès est refusé. |
| `404` | La ressource demandée n'a pas été trouvée. |
| Erreur de connexion | Le serveur n'a pas pu être contacté. |

✅ **À retenir :** HTTP permet aux applications clientes et aux serveurs web d'échanger des requêtes et des réponses. Le code de la réponse te dit comment ça s'est passé.

> [!TIP]
> 🔎 *Optionnel :* `man getent`, `man curl`. Spécifications : RFC 1034 et 1035 (DNS), RFC 9110 (HTTP).

---

## 2️⃣ Couche Transport

🎯 **Objectif :** observer les sockets TCP et UDP présents sur ta machine.

### Exercice 3 — Observer les sockets TCP

🧪 **Commande :**

```bash
ss -lnt
```

| Option | Rôle |
|---|---|
| `-l` | Affiche les sockets en écoute. |
| `-n` | Affiche les adresses et ports sous forme numérique. |
| `-t` | Affiche les sockets TCP. |

👀 **Résultat possible :**

```text
State   Recv-Q  Send-Q  Local Address:Port
LISTEN  0       4096    0.0.0.0:22
LISTEN  0       511     0.0.0.0:80
```

👁️ **Regarde seulement :** `LISTEN` et la colonne `Local Address:Port`.

🧠 **Comment comprendre le résultat ?**

- `LISTEN` : un service attend des connexions TCP.
- `0.0.0.0:22` : un service écoute sur le port 22, sur toutes les interfaces IPv4.
- `0.0.0.0:80` : un service écoute sur le port 80, sur toutes les interfaces IPv4.

Tu peux voir d'autres ports ou moins de lignes : cela dépend des services actifs sur ta machine. La liste peut même être vide.

✅ **À retenir :** un service qui attend des connexions TCP apparaît avec l'état `LISTEN`.

### Exercice 4 — Observer les sockets UDP

🧪 **Commande :**

```bash
ss -lnu
```

👀 **Résultat possible :**

```text
State   Recv-Q  Send-Q  Local Address:Port
UNCONN  0       0       127.0.0.53:53
```

👁️ **Regarde seulement :** l'état (`UNCONN`) et le port.

🧠 **Comment comprendre le résultat ?**

- `UNCONN` : socket UDP sans connexion établie.
- `127.0.0.53:53` : ici, un service DNS local sur le port 53.
- **Pourquoi `UNCONN` et pas `LISTEN` ?** UDP n'établit pas de connexion avant d'envoyer des données. Il est donc normal de voir `UNCONN`.

✅ **À retenir :** TCP et UDP sont deux protocoles de transport, mais ils ne gèrent pas les communications de la même manière.

> [!TIP]
> 🔎 *Optionnel :* `man ss`. Spécifications : RFC 9293 (TCP), RFC 768 (UDP).

---

## 3️⃣ Couche Internet

🎯 **Objectif :** identifier les adresses IP de ta machine et comprendre comment elle choisit une route.

### Exercice 5 — Observer les adresses IP

🧪 **Commande :**

```bash
ip addr
```

👀 **Résultat possible :**

```text
1: lo: <LOOPBACK,UP,LOWER_UP>
    inet 127.0.0.1/8

2: enp3s0: <BROADCAST,MULTICAST,UP>
    link/ether 00:11:22:33:44:55

3: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.20/24
    inet6 fe80::1234:5678:abcd:1234/64
```

Les noms et les adresses sont des exemples.

👁️ **Regarde seulement :** le nom de chaque interface, et les lignes `inet` (IPv4) et `inet6` (IPv6).

🧠 **Comment comprendre le résultat ?**

| Élément | Signification |
|---|---|
| `lo` | Interface loopback : la machine communique avec elle-même. |
| `enp3s0` | Exemple d'interface Ethernet. |
| `wlp2s0` | Exemple d'interface Wi-Fi. |
| `inet` / `inet6` | Une adresse IPv4 / une adresse IPv6. |
| `link/ether` | L'adresse MAC de l'interface. |

Trois états à connaître :

| État | Signification |
|---|---|
| `UP` | L'interface est activée. |
| `LOWER_UP` | La liaison réseau est opérationnelle. |
| `NO-CARRIER` | Aucune liaison détectée (par exemple, aucun câble Ethernet branché). |

✅ **À retenir :** `ip addr` te dit quelles adresses IP sont attribuées aux interfaces.

### Exercice 6 — Observer les interfaces réseau

🧪 **Commande :**

```bash
ip link
```

👀 **Résultat possible :**

```text
2: enp3s0: <BROADCAST,MULTICAST,UP>
    link/ether 00:11:22:33:44:55

3: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    link/ether aa:bb:cc:dd:ee:ff
```

👁️ **Regarde seulement :** le nom de l'interface, son état (`UP`) et la ligne `link/ether`.

🧠 **Comment comprendre le résultat ?**

| Commande | Ce qu'elle affiche |
|---|---|
| `ip addr` | Les interfaces et leurs **adresses IP**. |
| `ip link` | Les interfaces, leur **état** et leurs **adresses MAC**. |

✅ **À retenir :** une adresse IP sert à l'adressage au niveau Internet ; une adresse MAC identifie une interface au niveau de l'accès réseau.

### Exercice 7 — Comprendre la table de routage

🧪 **Commande :**

```bash
ip route
```

👀 **Résultat possible :**

```text
default via 192.168.1.1 dev wlp2s0
192.168.1.0/24 dev wlp2s0
```

👁️ **Regarde seulement :** la ligne qui commence par `default`.

🧠 **Comment comprendre le résultat ?**

| Élément | Signification |
|---|---|
| `default` | Route utilisée quand aucune route plus précise ne correspond. |
| `via 192.168.1.1` | La passerelle (souvent ta box ou ton routeur). |
| `dev wlp2s0` | L'interface utilisée pour envoyer les paquets. |

La deuxième ligne signifie que le réseau `192.168.1.0/24` est directement accessible par `wlp2s0`.

Chez toi, les adresses seront différentes (par exemple `default via 10.0.0.1 dev enp3s0`). La route par défaut peut aussi être absente si aucune passerelle n'est configurée.

✅ **À retenir :** la table de routage aide le système à choisir par quelle interface et quelle passerelle envoyer un paquet.

> [!TIP]
> 🔎 *Optionnel :* `man ip`, `man ip-route`. Spécification : RFC 791 (IPv4).

---

## 4️⃣ Couche Accès réseau

Cette couche concerne la transmission des données sur le réseau local, les trames Ethernet et les adresses MAC.

### Exercice 8 — Observer les voisins du réseau local

🧪 **Commande :**

```bash
ip neigh
```

👀 **Résultat possible :**

```text
192.168.1.1 dev wlp2s0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
```

👁️ **Regarde seulement :** l'adresse IP, l'adresse après `lladdr` (la MAC) et l'état à la fin.

🧠 **Comment comprendre le résultat ?**

- Ta machine connaît l'adresse MAC de ce voisin, et le considère comme joignable (`REACHABLE`).
- Tu peux aussi voir `STALE`, `DELAY` ou `INCOMPLETE`. La table peut être vide si aucun voisin n'a encore été découvert.

✅ **À retenir :** sur un réseau IPv4 local, ARP associe une adresse IP à une adresse MAC. IPv6 utilise NDP pour des fonctions équivalentes.

> [!TIP]
> 🔎 *Optionnel :* `man ip-neighbour`. Spécifications : RFC 826 (ARP), RFC 4861 (NDP).

---

# 🔬 Niveau 2 — Observer une communication TCP réelle

🎯 **Objectif :** créer un serveur HTTP local, établir une connexion TCP et repérer le handshake dans la capture.

Nous utilisons `127.0.0.1` : la communication reste sur ta machine.

> [!NOTE]
> Ouvre **trois terminaux** : un pour le serveur, un pour le client, un pour la capture.

## Étape 1 — Préparer le serveur

```bash
mkdir -p /tmp/tcpip-lab
echo "Hello TCP/IP" > /tmp/tcpip-lab/index.html
```

Aucune sortie n'est attendue si tout réussit.

## Étape 2 — Démarrer le serveur HTTP

**Terminal 1 :**

```bash
python3 -m http.server 8080 --bind 127.0.0.1 --directory /tmp/tcpip-lab
```

👀 **Résultat attendu :**

```text
Serving HTTP on 127.0.0.1 port 8080
```

Garde ce terminal ouvert.

## Étape 3 — Vérifier le port TCP

**Terminal 2 :**

```bash
ss -lnt
```

👀 **Résultat attendu :** une ligne avec le port `8080`, par exemple :

```text
LISTEN  0  5  127.0.0.1:8080  0.0.0.0:*
```

🧠 Le serveur attend des connexions TCP sur le port 8080, uniquement sur l'adresse loopback.

## Étape 4 — Démarrer la capture réseau

**Terminal 3 :**

```bash
sudo tcpdump -i lo -nn -A 'tcp port 8080'
```

| Élément | Rôle |
|---|---|
| `-i lo` | Capture sur l'interface loopback. |
| `-nn` | Garde les adresses et les ports sous forme numérique. |
| `-A` | Affiche les données capturées sous forme de texte lisible. |
| `tcp port 8080` | Ne garde que les paquets TCP du port 8080. |

Laisse la capture tourner.

## Étape 5 — Envoyer une requête HTTP

Dans le **terminal 2** :

```bash
curl http://127.0.0.1:8080
```

👀 **Résultat attendu :**

```text
Hello TCP/IP
```

Dans le terminal 1, le serveur affiche une ligne semblable à `127.0.0.1 - - "GET / HTTP/1.1" 200 -`. La requête a atteint le serveur, qui a répondu avec le code `200`.

## Étape 6 — Lire la capture (sans te noyer)

Dans le **terminal 3**, arrête la capture avec `Ctrl + C`.

La vraie sortie de `tcpdump` est longue : horodatages, numéros de séquence, tailles de fenêtre… **Tu peux ignorer tout ça.** Voici la même capture, réduite à ce qui compte (les `...` remplacent les détails que tu n'as pas besoin de lire) :

👀 **Exemple de capture (réduit à l'essentiel) :**

```text
IP 127.0.0.1.52114 > 127.0.0.1.8080: Flags [S],  ...
IP 127.0.0.1.8080 > 127.0.0.1.52114: Flags [S.], ...
IP 127.0.0.1.52114 > 127.0.0.1.8080: Flags [.],  ...
IP 127.0.0.1.52114 > 127.0.0.1.8080: Flags [P.], ...
GET / HTTP/1.1
IP 127.0.0.1.8080 > 127.0.0.1.52114: Flags [P.], ...
HTTP/1.0 200 OK
IP 127.0.0.1.8080 > 127.0.0.1.52114: Flags [F.], ...
IP 127.0.0.1.52114 > 127.0.0.1.8080: Flags [F.], ...
```

👁️ **Regarde seulement trois choses :**

1. **`Flags [...]`** : ce que fait chaque paquet.
2. **Le sens `>`** : qui parle à qui (client vers serveur, ou l'inverse). Le port `52114` est un port temporaire choisi par `curl` ; ton port sera différent.
3. **Les lignes de texte** `GET ...` et `HTTP/1.0 200 OK` : ce sont les données HTTP.

Tu n'as **pas** besoin de comprendre `seq`, `ack`, `win` ou `length` pour ce lab.

🧠 **Les flags à connaître :**

| Flag | Signification |
|---|---|
| `[S]` | Le client demande une connexion (SYN). |
| `[S.]` | Le serveur accepte (SYN-ACK). |
| `[.]` | Simple confirmation, sans données (ACK). |
| `[P.]` | Un côté envoie des données (PUSH + ACK) : c'est là que passent `GET` et la réponse. |
| `[F.]` | Un côté ferme la connexion (FIN). |

Les trois premiers paquets forment le **three-way handshake** : SYN, puis SYN-ACK, puis ACK. Ensuite seulement, le client envoie `GET / HTTP/1.1` et le serveur répond. Enfin, la connexion se ferme avec des paquets `FIN`.

> [!NOTE]
> Les détails varient d'une machine à l'autre. Le serveur de `python3 -m http.server` répond généralement en `HTTP/1.0`, alors que `curl` envoie sa requête en `HTTP/1.1`. C'est normal.

✅ **À retenir :** TCP établit une connexion (handshake) avant de transmettre les données de l'application.

> [!TIP]
> 🔎 *Optionnel :* `man tcpdump`, `man pcap-filter`. Spécification : RFC 9293.

---

# 📨 Niveau 3 — Observer une communication UDP

🎯 **Objectif :** envoyer un message UDP et le comparer à TCP.

## Étape 1 — Préparer un écouteur UDP

**Terminal 1 :**

```bash
nc -u -l 9999
```

Cette commande ouvre un écouteur UDP sur le port `9999`. Garde le terminal ouvert.

## Étape 2 — Capturer les paquets UDP

**Terminal 2 :**

```bash
sudo tcpdump -i lo -nn 'udp port 9999'
```

## Étape 3 — Envoyer un message

**Terminal 3 :**

```bash
echo "Hello UDP" | nc -u -w 1 127.0.0.1 9999
```

👀 **Résultat attendu dans le terminal 1 :**

```text
Hello UDP
```

👀 **Résultat attendu dans le terminal 2 (exemple) :**

```text
IP 127.0.0.1.41234 > 127.0.0.1.9999: UDP, length 10
```

👁️ **Regarde seulement :** le mot `UDP`, et ce qui **manque** : aucun `Flags`, aucun `[S]` ni `[S.]`. Un seul paquet suffit.

Arrête la capture avec `Ctrl + C`, puis ferme l'écouteur avec `Ctrl + C`.

🧠 **À comprendre :** contrairement à TCP, UDP n'effectue aucun three-way handshake. Le message part directement sous forme de datagramme.

### ⚖️ TCP et UDP en un coup d'œil

| Critère | TCP | UDP |
|---|---|---|
| Connexion préalable | Oui (three-way handshake) | Non |
| État dans `ss` | `LISTEN` | `UNCONN` |
| Dans la capture | `[S]`, `[S.]`, `[.]`, `[P.]`, `[F.]` | Un simple paquet `UDP` |

✅ **À retenir :** UDP permet d'envoyer des données sans établir de connexion au préalable.

> [!TIP]
> 🔎 *Optionnel :* `man nc`. Cherche quels usages réels correspondent à TCP (web, SSH, mail) et à UDP (DNS, streaming, jeux en ligne).

---

# 🛠️ Niveau 4 — Diagnostiquer une communication qui échoue

🎯 **Objectif :** comprendre comment une panne bloque une communication, et savoir **où** elle se situe.

Nous réutilisons le serveur HTTP local du niveau 2.

## Panne 1 — Le serveur n'est plus disponible

Arrête le serveur Python avec `Ctrl + C`, puis exécute :

```bash
curl http://127.0.0.1:8080
```

👀 **Résultat possible :**

```text
curl: (7) Failed to connect to 127.0.0.1 port 8080
```

Le message exact varie selon la version de `curl`.

🧠 La machine a toujours son adresse loopback, mais **aucun serveur n'écoute** sur le port 8080. La connexion TCP ne peut pas s'établir.

## Panne 2 — Le serveur répond, mais la ressource n'existe pas

Redémarre le serveur :

```bash
python3 -m http.server 8080 --bind 127.0.0.1 --directory /tmp/tcpip-lab
```

Dans un autre terminal :

```bash
curl -i http://127.0.0.1:8080/page-inexistante
```

👀 **Résultat attendu :** une réponse HTTP avec le code `404`.

🧠 La connexion TCP a réussi, le serveur a reçu la requête, mais la ressource n'existe pas : il renvoie `404 Not Found`.

### 🩺 Où se situe la panne ?

| Symptôme | Couche concernée | Ce que ça signifie |
|---|---|---|
| `curl: (7) Failed to connect` | Transport (TCP) | Personne n'écoute sur ce port : la connexion échoue. |
| `404 Not Found` | Application (HTTP) | La connexion fonctionne, mais la ressource n'existe pas. |

✅ **À retenir :** une connexion réseau réussie ne signifie pas que la ressource existe. Pour diagnostiquer, trouve d'abord la couche où ça casse.

> [!TIP]
> 🔎 *Optionnel :* refais la panne 1 avec `tcpdump` allumé et observe la réponse de ta machine quand personne n'écoute. Cherche ce qu'est un paquet `RST`.

---

# 🏁 Bilan du laboratoire

| Couche | Ce que tu as appris à observer | Outils utilisés |
|---|---|---|
| **Application** | Résolution de noms et réponses HTTP. | `getent`, `curl` |
| **Transport** | Sockets TCP/UDP, ports et handshake TCP. | `ss`, `tcpdump`, `nc` |
| **Internet** | Adresses IP et table de routage. | `ip addr`, `ip route` |
| **Accès réseau** | Interfaces, adresses MAC et voisins. | `ip link`, `ip neigh` |

---

# 🚀 Va plus loin : ne te limite pas à ce laboratoire

> [!IMPORTANT]
> Ce laboratoire est un **point de départ**, pas une destination. Il te donne l'essentiel pour observer le réseau, mais chaque commande et chaque protocole cache beaucoup plus de détails. Tu n'as pas à tout comprendre aujourd'hui : avance étape par étape.

Quand un point te semble flou, **cherche ailleurs**. C'est ce que font les administrateurs système au quotidien.

1. **Lis le manuel.** Chaque commande a une page `man` (`man ss`, `man ip`, `man tcpdump`). Essaie aussi `commande --help`.
2. **Change les options.** Relance une commande autrement et observe ce qui change.
3. **Va voir les sources officielles.** Les RFC sont les textes de référence : RFC 9293 (TCP), RFC 768 (UDP), RFC 791 (IP), RFC 826 (ARP), RFC 4861 (NDP), RFC 1034 et 1035 (DNS), RFC 9110 (HTTP).
4. **Croise les sources.** Une explication différente peut débloquer ta compréhension.
5. **Casse et répare.** Change un port, arrête un service, puis explique pourquoi le résultat a changé.
6. **Demande-toi « pourquoi ? »** Ne retiens pas seulement ce que fait une commande : cherche pourquoi le protocole fonctionne ainsi.

**Pistes pour la suite :**

- Pourquoi TCP utilise-t-il un handshake en trois étapes plutôt que deux ?
- Que se passe-t-il quand un paquet TCP est perdu ?
- Comment fonctionne le DNS en détail ?
- Comment fonctionnent NAT et les adresses privées ?
- Que montre une capture d'une requête HTTPS, et pourquoi le contenu est-il illisible ?

> [!NOTE]
> Tu as trouvé une erreur ou une commande qui ne fonctionne pas sur ta machine ? Ouvre une *Issue* sur ce dépôt en précisant ta distribution et la commande concernée.

---

**HEAVYLOCK — Build locks. Learn how they break!**
