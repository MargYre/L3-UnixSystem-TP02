# Préparation avant le début du TP 02
Tout d'abord, j'ai rencontré des difficultés avec la VM, ce qui m'a contraint à la désinstaller puis à la réinstaller. J'ai également dû libérer de l'espace dans mon répertoire /home/tmargaux afin de permettre à la VM de fonctionner correctement.
Durant plusieurs heures pendant le TP en cours, je n'arrivais pas à établir une connexion SSH entre ma machine hôte et ma VM. Après avoir réinstallé ma VM, j’ai finalement réussi à me reconnecter. 
Cependant, la raison pour laquelle cela n'a pas fonctionné plus tôt me reste inconnue (les mystères de l'informatique...). 

Installation du serveur SSH :
```bash
apt install openssh-server -y
```
Modification du fichier sshd_config pour permettre les connexions root :
```bash
nano /etc/ssh/sshd_config
PasswordAuthentication yes
```
Redémarrage du service :
```bash
systemctl restart ssh
```
Connexion SSH à la VM :
```bash
ssh root@localhost -p 2222
```
# TP 02 : Services, processus signaux

## 1 Secure Shell : SSH
### 1.1 Exercice : Connection ssh root (reprise fin tp-01)
```bash
man sshd_config:
```
Le fichier /etc/ssh/sshd_config contient des paires clé-argument pour configurer SSH. Voici les éléments modifiés et leurs options :
**-PermitRootLogin:** Spécifie si root peut se connecter en utilisant ssh(1).
yes : Autorise la connexion root avec mot de passe.
prohibit-password : Désactive le mot de passe et n'autorise que les connexions par clé publique.
forced-commands-only : Autorise root à se connecter par clé publique pour des commandes spécifiques.
no : Interdit les connexions root.
**Avantages/Inconvénients :**
yes : Simple mais moins sécurisé.
prohibit-password : Plus sûr car il empêche l'utilisation de mots de passe.
no : La méthode la plus sécurisée.
**-PasswordAuthentication:** Spécifie si l'authentification par mot de passe est autorisée. La valeur par défaut est oui.

### 1.2 Exercice : Authentification par clef / Génération de clefs
Ici nous n’utiliseront pas de passphrase pour simplifier les choses, C’est une mauvaise idée dans un cas réel car: Une phrase secrète sécurisée permet d'empêcher la copie et l'utilisation de votre clé privée même si votre ordinateur est compromis.
**Etapes:**
	```bash
	-ssh-keygen
	```
	-Enter file in which to save the key (/root/.ssh/id_rsa)
	-Pas de passphrase
	-Your identification has been saved in /root/.ssh/id_rsa/Your public key has been saved in /root/.ssh/id_rsa.pub
	-Copie d'une clé publique SSH sur votre serveur: 
	```bash
	ssh-copy-id root@localhost -p 2222
	```
Normalement la clé publique est correctement installée sur le serveur.

*sources:* https://www-inf.telecom-sudparis.eu/cours/UNIX/Web/9.63.21.html
https://learn.microsoft.com/en-us/azure/devops/repos/git/gcm-ssh-passphrase?view=azure-devops
https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server

### 1.3 M. Le Cocq . 2024 Exercice : Authentification par clef / Connection serveur

J'ai déjà réaliser la connexion par clé publique dans la partie précdente.
J'ai utilisé la commande suivante pour tester la connexion :
```bash
-ssh -p '2222' 'root@localhost' ? EN fait j’ai fais la part 3 dans la part 2.
```
La connexion s'est bien établie, confirmant que la clé publique est bien installée.

### 1.4 Exercice : Authentification par clef : depuis la machine hote
```bash
ssh -i id_rsa.pub root@localhost -p 2222
```
La connexion s'est établie sans demander de mot de passe.

### 1.5 Exercise: Sécurisez
**1. Switch to SSH key authentication:**
```bash
sudo nano /etc/ssh/sshd_config
PasswordAuthentication no
sudo systemctl reload ssh

```

**Qu'est-ce qu'une attaque par force brute SSH ?**
Une attaque par force brute (bruteforce attack) consiste à tester, l'une après l'autre, chaque combinaison possible d'un mot de passe ou d'une clé pour un identifiant donné afin se connecter au service ciblé. Il s'agit d'une méthode ancienne et répandue chez les pirates.

**Techniques permettant de réduire les risques liés aux attaques par force brute SSH :**

1. Limiter les tentatives d'authentification:
	1. Limiter les tentatives d'authentification: 
	Pour protéger davantage SSH contre les attaques par force brute on peut limiter le nombre de tentatives d'authentification.Cela peut être accompli en ajustant la variable MaxTries dans votre fichier sshd_config.
2. Mettre en place l'authentification à deux facteurs (2FA):
	L'intégration de l'authentification à deux facteurs (2FA) est un moyen simple et efficace d'améliorer la sécurité et de protéger SSH contre les attaques par force brute. Le module Google Authenticator PAM est un choix populaire pour ce cas d'utilisation.

Chaque méthode présente des *avantages* : l'authentification par clé SSH élimine les risques liés aux mots de passe, tandis que la restriction des utilisateurs limite les accès non autorisés. Cependant, ces méthodes nécessitent une *gestion attentive* des clés et des comptes pour éviter des verrouillages accidentels du serveur.
*sources:* https://goteleport.com/blog/ssh-hardening-to-prevent-brute-force-attacks/,https://blog.sucuri.net/2023/04/how-to-prevent-ssh-brute-force-login-attacks.html,
SSH security best practices non-root users, https://www.blumira.com/blog/secure-ssh-on-linux (surtotut blog.sucuri.net et l'article "How to Prevent SSH Brute Force Login Attacks")


## 2 Processus
### 2.1 Exercice : Etude des processus UNIX
Je vais répondre aux questions suivantes à l'aide du man de ps.
**A quoi correspond l’information TIME ?** temps CPU cumulé, "[DD-]HH:MM:SS"
                               format.  (alias CPUtime).
En gros ça correspond au temps CPU total utilisé par le processus depuis son démarrage.
**Quel est le processus ayant le plus utilis´e le processeur sur votre machine ?**
```bash
ps aux --sort -%cpu
```
Affiche tous les processus et les trie par utilisation CPU, du plus élevé au plus bas.

*sources:* https://www.atlantic.net/vps-hosting/find-top-10-running-processes-by-memory-and-cpu-usage/

## 3 Exercice 2 : Arrˆet d’un processus
**man date :**
Expliquant le formatage de la date : +%T affiche l'heure au format HH:MM
*--date :* Explique comment manipuler les dates en utilisant une durée relative, comme dans --date '5 hour ago'.
**man sleep :**
Décrit comment la commande met en pause l'exécution du script pendant un nombre de secondes spécifié (dans ce cas, sleep 1 met en pause pour 1 seconde).

**Que font les scripts ?** Les scripts utilisent une boucle infinie pour afficher la date toutes les secondes. La commande sleep 1 pause l'exécution du script pendant 1 seconde entre chaque affichage. Dans date.sh, la date est affichée au format HH:MM avec date +%T. Dans date-toto.sh, la date affichée est celle d'il y a 5 heures avec date --date '5 hour ago' +%T.

## 4 Exercice 3 : les tubes
**Quelle est la diff´erence entre tee et cat ?** cat et tee se comportent de la même manière lorsque vous ne leur donnez aucun nom de fichier. Cat lit potentiellement de nombreux fichiers et envoie leur sortie à un seul endroit, tandis que tee lit une entrée et l'envoie potentiellement à plusieurs fichiers.
En résumé, cat sert à la lecture (saisie manuelle et fichiers) et à l'écriture (avec > et >>) dans des fichiers. Et tee consiste uniquement à écrire dans des fichiers - mais simultanément en fonction de la sortie de la commande qui sert à son entrée.

```bash
$ ls | cat
```
Affiche la liste des fichiers comme si on exécutait ls seul
```bash
$ ls -l | cat > liste

```
Affiche les fichiers avec détails (ls -l) et enregistre la sortie dans le fichier liste
```bash
$ ls -l | tee liste

```
 Affiche les fichiers avec détails et sauvegarde la sortie dans liste en même temps
```bash
$ ls -l | tee liste | wc -l

```
Sauvegarde la sortie de ls -l dans liste et compte le nombre de lignes (fichiers)

*sources:* https://askubuntu.com/questions/1152444/what-is-the-difference-between-cat-and-tee, le man

## 5 Journal syst`eme rsyslog

**A quoi sert le service cron ?** 
Cron est un programme qui permet aux utilisateurs des systèmes Unix d'exécuter automatiquement des scripts, des commandes ou des logiciels à une date et une heure spécifiée à l'avance, ou selon un cycle défini à l'avance.
**Que fait la commande tail -f ?**
tail - affiche la dernière partie des fichiers
-f, --follow[={nom|descripteur}]
              afficher les données ajoutées à mesure que le fichier grandit ;

**/etc/logrotate :** Ce fichier de configuration contient les directives sur la manière dont les fichiers journaux doivent être alternés par défaut. S'il n'existe pas d'ensemble spécifique de directives, l'utilitaire agit conformément aux directives de ce fichier. Vous trouverez ci-dessous un exemple du contenu du fichier de configuration.

**dmesg:**dmesg (pour l'anglais "display message", "afficher message" en français) est une commande sur les systèmes d'exploitation de type Unix qui affiche la mémoire tampon de message du noyau. Quand le système d'un ordinateur est amorcé, le noyau est chargé en mémoire.

*sources:* le man, https://www.redhat.com/en/blog/setting-logrotate#:~:text=%2Fetc%2Flogrotate.&text=conf%20.,contents%20of%20the%20configuration%20file, wikipédia
