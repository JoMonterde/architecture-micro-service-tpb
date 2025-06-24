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
### 1. Quelle est la structure de la ligne de requête HTTP reçue côté serveur ? Quels sont les trois éléments que vous devez extraire ?
On a la structure suivante : GET /motd HTTP/1.1
On a donc à extraire : 
1. Méthode (l'action)
2. Chemin (l'accès à la ressource)
3. Version (la version du protocole)

### 2. Que se passe-t-il si la ligne est mal formée (ex. : vide ou incomplète) ? Comment éviter une erreur dans votre code ?
Si la ligne est mal formée, cela provoquera une exception (ValueError)

### 3. Que contient exactement la variable chemin ? Est-ce toujours ce que l’utilisateur a tapé dans l’URL ?
La variable chemin contient le chemin brut de l’URL demandée.

### 4. Que fait votre serveur si la méthode reçue est autre que GET ? Par exemple : POST, FOO, TEAPOT ?
Deux cas possibles : soit les méthodes sont traitées par le serveur, auquel cas on aura un fonctionnement spécifique selon la situation (POST permettra de modifier les données, par exemple).
Par contre, si notre serveur ne sait traiter que la méthode GET, le reste doit être ignoré ou renvoyer une erreur.
### 5. Que se passe-t-il si le chemin n’est pas reconnu (ex. : /truc) ? Quel message retournez-vous ? Est-il lisible ?
On retourne une erreur 404 Not Found. Le message est en soit lisible.

### 6. Le message 404 Not Found est-il suffisant comme réponse ? Pourriez-vous y ajouter un texte plus explicite ?
Il peut suffire en tant que tel, mais il peut effectivement être personnalisé, avec un texte plus explicite selon les cas, pour aider l'utilisateur.

### 7. Est-ce que les chemins /motd, /motd/, et /motd?x=42 sont identiques ? Comment les différencier ou les normaliser ?
Non, ils sont différents :
- /motd : chemin de base.
- /motd/ : peut être traité comme un répertoire ou un autre chemin.
- /motd?x=42 : inclut des paramètres de requête.

### 8. Pour ajouter un nouveau chemin comme /bonjour, que faut-il modifier dans votre code ? Est-ce que votre structure est facilement extensible ?
> Il suffit d’ajouter une nouvelle branche de condition dans notre code. C'est assez facilement extensible, mais cette structure peut vite devenir difficilement maintenable. Utiliser un dictionnaire serait plus propre.

### 9. Que pourrait-il se passer si un utilisateur tape une URL comme /../../etc/passwd ? Pourquoi est-ce potentiellement dangereux ?
Le mot de passe n'est pas du tout protégé, il peut être très facilement intercepté. Cela est donc dangereux car pas du tout protégé.

### 10. Est-ce que vous traitez le chemin comme une simple chaîne de caractères, ou comme une ressource contrôlée ? Quelles protections pourriez-vous ajouter pour éviter les comportements inattendus ?
Si vous traitez le chemin comme une chaîne de caractères brute, vous êtes vulnérable.
Il faut donc protéger les URL pour éviter l'injection de code, notamment. 

## 2.3 Serveur dynamique, avec /date

### 1. Quelle est la ligne exacte reçue par le serveur quand un client appelle /date ? Comment repérez-vous ce chemin ?
La première ligne reçue est :

sql
Copier
Modifier
GET /date HTTP/1.1

### 2. Comment déterminez-vous que la requête est terminée ? Quelle est l’utilité de la ligne vide à la fin de la requête ?
Justement grâce à la présence de cette ligne vide, qui sépare en-tête et corps (de la requête tout court, dans le cas d'un GET qui n'a donc pas de body).

### 3. Que contient la réponse retournée par votre serveur ? Est-ce que la date change à chaque appel ?

Nous sommes le lundi 24 juin 2025, 14:18:02
Le corps contient la date et l’heure actuelles, donc la valeur change à chaque appel, grâce à datetime.

### 4. Quel champ d’en-tête est indispensable pour que la réponse soit bien comprise par le client ? Que se passe-t-il si vous oubliez Content-Length ?
Le champ Content-Length est indispensable, car certains clients (navigateur, curl) peuvent ne rien afficher du tout ou attendre la fin de la connexion. D’autres peuvent tronquer ou mal interpréter le contenu.

### 5. Que se passe-t-il si Content-Length est incorrect (trop petit ou trop grand) ? Testez avec curl, telnet, et un navigateur.
Trop petit : une partie du message est coupée.

Trop grand : le client peut : attendre indéfiniment le reste des données (curl, telnet), afficher partiellement ou se bloquer (navigateur), générer une erreur de format.

### 6. Le corps de votre réponse contient-il des caractères spéciaux ou des retours à la ligne (\n) ? Comment sont-ils affichés selon le client utilisé ?
Navigateur : rend les \n en texte brut, pas de saut de ligne (sauf avec <br> en HTML).

curl / telnet : affichent bien les sauts de ligne dans le terminal.

### 7. Que renvoie votre serveur si un autre chemin est demandé, par exemple /time ou /now ? Est-ce prévu ? Que se passe-t-il ?
Ce n'est pas prévu, donc le serveur retourne une réponse 404.

### 8. Est-ce que le serveur répond de manière fiable même si deux clients se connectent successivement ? Et en même temps ?
Successivement : oui, aucun problème, les requêtes sont traitées une par une.

En même temps : pour répondre à plusieurs clients en parallèle, il faut utiliser des threads ou un serveur asynchrone, sinon cela génère des problèmes.

### 9. Pourriez-vous ajouter un autre chemin dynamique (comme /uptime) ? Quelles informations pourraient être utiles à afficher ?
Oui, c'est envisageable. On pourrait par exemple afficher le temps écoulé depuis le lancement du serveur, l'adresse IP du client, le nombre de connexions, ...

### 10. Que faudrait-il faire pour rendre ce serveur un peu plus “propre” ou modulaire ? Est-ce que le code pourrait être réorganisé pour gérer plusieurs routes dynamiques plus facilement ?
Oui. Pour une meilleure maintenabilité, un dictionnaire serait une bonne idée.
