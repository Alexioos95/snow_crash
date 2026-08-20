# snow_crash

Introduction aux risques simples d'un mauvais réglage des permissions de fichiers et de codes vulnérables, à travers de petits exercices de recherche de failles de sécurité via un terminal.

Le projet s'est effectué dans une VM minutieusement configurée par l'école, dans laquelle l'élève doit rechercher la faille avec son utilisateur actuel et récupérer le mot de passe du suivant. Un compte rendu est écrit dans le ```/resources/explanation.md``` de chaque niveau.

Failles de chaque niveau:

0. Chiffrement par code César
1. Hachage DES et John The Ripper
2. Paquets réseau et Wireshark
3. Variables d'environnement
4. CGI - Paramètre de query string
5. Crontab et injection de commande
6. preg_replace /e de PHP et substitution pour injection de commande
7. Variables d'environnement et échappement pour injection de commande
8. Liens symboliques
9. Chiffrement décalage progressif
10. Time-of-check to time-of-use (TOCTOU)
11. Échappement d'un serveur de chat en direct pour injection de commande
12. CGI - Injection de commande
13. Ghidra
14. Ghidra et ASM

Crédits:  
[Alexis Payen](https://github.com/Alexioos95/) - Recherche de failles et écriture des comptes rendus.  
[Eli Ewu](https://github.com/Uweile) - Recherche de failles et relecture des comptes rendus.
