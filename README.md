Introduction aux risques simples d'un mauvais réglage des droits de fichiers et de code non sécurisé, à travers de petits exercices de recherche de failles de sécurité via un terminal.

Le projet s'est effectué dans une VM minutieusement organisée par l'école, où l'élève doit chercher la faille avec son user actuel, et récupérer le mot de passe du suivant. Un compte rendu est écrit dans le ```/resources/explanation.md``` de chaque niveau.

Failles de chaque niveau:

0. Chiffrement code César
1. DES hash et JohnTheRipper
2. Packet et WireShark
3. Variables d'environnement
4. CGI - Paramètre query string
5. Crontab et injection de commande
6. PHP's preg_replace /e et substitution pour injection de commande
7. Variables d'environnement et échappement pour injection de commande
8. Liens symboliques
9. Chiffrement décalage progressif
10. Time-of-check to time-of-use
11. Echappement d'un serveur chat pour injection de commande
12. CGI - Injection de commande
13. Ghidra
14. Ghidra et ASM

Crédits:  
[Alexis Payen](https://github.com/Alexioos95/) - Recherche de failles et écriture des comptes-rendus.  
[Eli Ewu](https://github.com/Uweile) - Recherche de failles et relecture des comptes-rendus.
