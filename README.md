# DevOps infrastructure defense guide

Guide personnel pour comprendre et présenter en 20 minutes une migration d'infrastructure :

1. audit et changement de VPS ;
2. Docker Swarm avec Traefik ;
3. diagnostic d'incidents ;
4. migration de l'edge vers Nginx et Certbot ;
5. sécurité, secrets, sauvegarde et reprise ;
6. trajectoire K3s avec observabilité.

Le but n'est pas de réciter des définitions. Il faut raconter une décision d'ingénierie : **problème → hypothèses → preuves → choix → limites → prochaine étape**.

## Parcours recommandé

| Ordre | Fichier | Objectif |
|---:|---|---|
| 1 | [Plan de 20 minutes](00-plan-20-minutes.md) | Comprendre le rythme et le message de chaque partie. |
| 2 | [Script oral](01-script-oral.md) | S'entraîner avec des formulations directement prononçables. |
| 3 | [Architecture globale](02-architecture-globale.md) | Visualiser les trois phases sans diagramme illisible. |
| 4 | [Docker Swarm et Traefik](03-swarm-traefik.md) | Maîtriser l'architecture initiale. |
| 5 | [Nginx, DNS et TLS](04-nginx-dns-tls.md) | Expliquer la cible technique en profondeur. |
| 6 | [Sécurité et reprise](05-securite-sauvegarde.md) | Défendre les choix de sécurité et de backup. |
| 7 | [K3s et observabilité](06-k3s-observabilite.md) | Présenter la trajectoire sans prétendre qu'elle est déjà en production. |
| 8 | [Validation et GitOps](07-validation-gitops.md) | Montrer comment les erreurs sont bloquées avant le déploiement. |
| 9 | [Questions du jury](08-questions-jury.md) | Préparer les objections et réponses courtes. |
| 10 | [Vocabulaire et phrases fortes](09-vocabulaire.md) | Parler avec précision, sans buzzwords inutiles. |
| 11 | [Commandes à expliquer](10-commandes-demo.md) | Savoir lire les commandes, même sans démonstration live. |

## Les cinq idées à retenir

- Un orchestrateur sur un seul nœud apporte un **état désiré**, pas une haute disponibilité physique.
- Nginx n'est pas « meilleur » que Traefik : il était **mieux aligné avec un catalogue de routes stable et une petite équipe**.
- Une image épinglée par digest garantit son identité, pas l'absence de vulnérabilités.
- Une sauvegarde n'est fiable qu'après une restauration testée.
- Le dépôt K3s est une baseline de laboratoire validée statiquement, pas une production déjà prouvée.

## Dépôts techniques associés

- [swarm-traefik-platform](https://github.com/AnouarMohamed/swarm-traefik-platform) : architecture Swarm initiale et découverte dynamique.
- [swarm-nginx-edge](https://github.com/AnouarMohamed/swarm-nginx-edge) : edge explicite, TLS, routage et validation.
- [k3s-application-platform](https://github.com/AnouarMohamed/k3s-application-platform) : cible Kubernetes légère, sécurité et observabilité.
