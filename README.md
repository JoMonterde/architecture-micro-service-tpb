Travaux réalisés par Jodie Monterde & Romain Courbaize

# architecture-micro-service-tpb

# 1 Lire une réponse HTTP avec telnet
## 1.5 Questions intermédiaires
### Quelle est la structure de la ligne GET / HTTP/1.1 ? Quels sont les trois éléments et leur rôle ?
La méthode permet de définir l'action (GET pour une récupération) - L'URL est le chemin vers la destination (/) - La version du protocole (HTTP/1.1)
### Pourquoi faut-il appuyer deux fois sur Entrée pour que le serveur réponde ? Que représente cette ligne vide dans le protocole HTTP ?
Il faut appyer deux fois pour marquer une ligne de séparation de l'entrée du contenu (par exemple dans le cadre d'un POST).
La ligne vide représente la séparation entre l'entête et le body.
### Que se passe-t-il si vous oubliez cette ligne vide ? Le serveur répond-il quand même ?
La ligne vide est obligatoire pour informer que l'entête est terminée. Le serveur ne répond pas.
### Que signifie le code 200 OK dans la réponse ? Qu’indique un code 404 ou 301 ?
200 -> Récupération réussie
404 -> Le serveur n'a pas trouvé les ressources demandées (erreur côté client)
301 -> L'URL demandé a été déplacée
### Quelle est la toute première ligne envoyée par le serveur ? Que contient-elle ?
HTTP/1.1 200 OK\
Il renvoie le code de retour HTTP.
### Relevez les entêtes présents dans la réponse (Content-Type, Content-Length, etc.) : à quoi servent-ils ?
Content-Type: text/html\
Connection: keep-alive\
Transfer-Encoding: chunked\
Last-Modified: Thu, 02 Jun 2016 ...
Chacun à son rôle, il spécifie la manière dont est traitée la réponse ou donne des informations. Par exemple, on sait que le contenu est de type text/html.
### Quelle est la première ligne non entête du corps ? À quoi reconnaît-on la séparation ?
_cc_ Normalement on devrait retrouver la première balise html.
Une ligne vide permet de faire la séparation entre l'entête et le body.
### Que se passe-t-il si vous tapez une URL invalide (ex : GET /truc HTTP/1.1) ? Quel est le code de retour ?
Une réponse est envoyé et informe que la page n'existe pas avec un code retour _404 Not found_.
### Le contenu retourné dans ce cas est-il aussi du HTML ? Que contient-il exactement ?
Un contenu HTML est retourné informant que la page n'existe pas (titre, message et code d'erreur).

# 2 Implémenter un mini-serveur HTTP (TCP brut)
## 2.1 Serveur minimal - “Hello World”
### Est-ce que votre client (navigateur, curl, telnet…) reçoit bien une réponse ? Quel contenu voyez-vous à l’écran ?
Oui, il reçoit bien la réponse, on peut voir _Hello world !_ à l'écran.
### Quelle est la structure exacte du message que vous renvoyez (statut, en-têtes, ligne vide, corps) ?
La réponse a un code 200 OK
_HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 12
Connection: close
2.1 Serveur minimal - “Hello World”
Hello World!_
### Que se passe-t-il si vous changez Content-Length: 12 en une autre valeur ?
Le message est tronqué à la dimension indiqué dans _Content-Length_ si cette dernière est inférieure à la longueur du message.
### Que se passe-t-il si vous omettez complètement cette ligne ?
Le client calculera lui-même la longueur de ce qu'il doit afficher.
### Que se passe-t-il si vous supprimez la ligne vide entre les entêtes et le message ? Le navigateur ou le client affiche-t-il encore quelque chose ?
Cela n'affiche rien : on considère que sans séparation, il n'y a pas de body donc rien à afficher.
###  Le message s’affiche-t-il dans telnet ? Et dans curl ? Et dans un navigateur ? Pourquoi certains outils sont-ils plus stricts que d’autres ?
Le message s'affiche dans telnet, dans curl et dans un navigateur. Il ne s'affiche cependant pas de la même manière. Curl et le navigateur n'affichent que _Hello world !_ tandis que telnet va renvoyer toute la réponse.\
Le protocole HTTP est plus strict dans certains cas.

## 2.2 Serveur avec analyse du chemin
### — Quelles sont les grandes responsabilités fonctionnelles de votre application serveur (gestion client, traitement commande, envoi message, logs, persistance…) ?
On peut imaginer avoir :
- la gestion des connexions clients
- l'analyse et exécution des commandes
- la diffusion des messages
### Peut-on tracer une frontière claire entre logique métier et logique d’entrée/sortie réseau ?
On peut distinguer la logique métier (traitement des commandes, gestion des canaux) de la logique réseau (écoute socket, send/recv).
### En cas d’erreur dans une commande, quelle couche doit réagir ?
La couche métier (l'analyse et exécution des commandes) doit signaler l’erreur. Et la couche réseau peut se charger de transmettre le message d’erreur au client.

## 2.3 Scalabilité et capacité à évoluer
### Si vous deviez ajouter une nouvelle commande (ex : /topic, /invite,/ban), quelle partie du système est concernée ?
Pour ajouter des commandes, il faut modifier l'analyse et l'exécution des commandes.
### Que faudrait-il pour que ce serveur fonctionne à grande échelle (plusieurs centaines de clients) ?
Il faudrait distribuer l'application sur plusieurs serveurs.
### Quelles limitations structurelles du code actuel empêchent une montée en charge ?
Il n'y a pas d'espace partagé et de lock.

## 2.4 Portabilité de l’architecture
### Ce serveur TCP pourrait-il être adapté en serveur HTTP ? Quelles parties seraient conservées, quelles parties changeraient ?
On garde la partie métier et on change toute la partie réseau.
### Dans une perspective micro-services, quels modules seraient candidats naturels pour devenir des services indépendants ?
Il y aurait :
- Authentification
- Gestion des canaux
- Messages & gestion commande
### Est-il envisageable de découpler la gestion des utilisateurs de celle des canaux ? Comment ?
On peut découpler l'ensemble en utilisant des APIs qui feront office de services.

## 2.5 Fiabilité, tolérance aux erreurs, robustesse
### Le serveur sait-il détecter une déconnexion brutale d’un client ? Peut-il s’en remettre ?
Il faut qu'il try les exceptions des sockets et donc faire le nécessaire dans ce cas.
### Si un message ne peut pas être livré à un client (socket cassée), le système le détecte-t-il ?
La socket lève une erreur.
### Peut-on garantir une livraison ou au moins une trace fiable de ce qui a été tenté/envoyé ?
On peut imaginer d'avoir un historique de tous les messages envoyés.

## 2.6 Protocole : structuration et évolutivité
Quelles sont les règles implicites du protocole que vous utilisez ? Une ligne = une commande, avec un préfixe (/msg, /join, etc.) et éventuellement des arguments : est-ce un protocole explicite, documenté, formalisé ?
