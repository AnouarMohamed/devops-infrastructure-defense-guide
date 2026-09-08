# Script oral — version 20 minutes

Ne pas apprendre chaque mot. Apprendre le **message**, les termes en gras et la première phrase de chaque section.

## 1. Ouverture — 45 secondes

> Bonjour. Mon projet de fin d'année porte sur la migration et la stabilisation d'une infrastructure web multi-services. Mon travail n'a pas consisté uniquement à déplacer des conteneurs d'un VPS vers un autre. J'ai traité l'infrastructure comme un système complet : hébergement, orchestration, reverse proxy, TLS, exposition réseau, secrets, sauvegardes, observabilité et reprise après incident. Le fil conducteur de mon projet est simple : rendre l'infrastructure plus lisible, plus déterministe et plus récupérable.

Phrase forte :

> **Une infrastructure n'est pas fiable parce qu'elle démarre ; elle est fiable lorsque son état est vérifiable et que sa reprise est démontrable.**

## 2. Contexte et problématique — 1 minute 15

> L'environnement hébergeait plusieurs familles de services : des sites WordPress, GLPI, Superset, Jenkins, Portainer, Passbolt et plusieurs bases de données. Tous partageaient un VPS aux ressources limitées. Cela crée trois contraintes. Premièrement, une saturation CPU, mémoire ou disque peut affecter plusieurs services simultanément. Deuxièmement, une mauvaise exposition réseau peut rendre une interface d'administration ou une base accessible directement. Troisièmement, un nœud unique concentre le risque de panne et impose une vraie stratégie de sauvegarde hors machine.

> Ma problématique était donc la suivante : comment migrer cette plateforme, stabiliser son point d'entrée et préparer son évolution, tout en restant proportionné à la taille de l'équipe et aux ressources disponibles ?

Ne pas dire : « Le serveur était mauvais. »

Dire : « Le serveur présentait une marge de capacité et d'exploitation insuffisante pour la trajectoire prévue. »

## 3. Audit et choix du VPS — 1 minute 15

> J'ai commencé par un inventaire : services, ports, volumes, domaines, bases, dépendances et données persistantes. Ensuite, j'ai comparé plusieurs hébergeurs selon six critères : calcul, mémoire, stockage, réseau, fonctions d'exploitation et coût total. Le prix n'était qu'un critère parmi les autres.

> Le choix s'est porté sur un VPS Hetzner qui proposait, au moment de l'étude, un meilleur équilibre entre CPU, stockage NVMe, trafic inclus et possibilités d'automatisation. Je précise que ce choix augmente la marge mais ne corrige pas une mauvaise architecture. La migration a donc été planifiée comme un transfert d'état : sauvegarder les bases et volumes, préparer la cible, tester avec la résolution locale, basculer le DNS, observer, puis conserver un retour arrière.

Phrase forte :

> **On ne migre pas des conteneurs ; on migre des données, des dépendances et des contrats de service.**

## 4. Première architecture : Swarm et Traefik — 2 minutes 15

> La première cible utilisait Docker Swarm. Swarm apporte un état désiré : je déclare un service, son image, ses réseaux, ses secrets et sa politique de redémarrage ; le manager essaie de faire converger l'état réel vers cette déclaration. Les réseaux overlay permettent au reverse proxy de joindre les services par leur nom, sans publier chaque port sur l'hôte.

> Traefik assurait l'edge, c'est-à-dire le point d'entrée Internet. Son provider Docker observait les services Swarm et générait le routage à partir des labels. Pour publier une application, les labels définissaient le nom d'hôte, l'entrypoint HTTPS, le résolveur ACME, le port interne et les middlewares. Cette approche était efficace pour ajouter rapidement des services.

> Les middlewares centralisaient la redirection HTTPS, les en-têtes, l'authentification et la limitation de débit. L'option `exposedByDefault=false` était importante : un service n'était public que si son manifeste l'autorisait explicitement.

> Je dois cependant distinguer orchestration et haute disponibilité. Dans ce projet, les services reposaient principalement sur un seul VPS. Swarm améliorait la reproductibilité et les redémarrages, mais une panne du nœud ou du disque affectait toujours l'ensemble. **Un orchestrateur mono-nœud apporte de la convergence, pas de la redondance physique.**

> Enfin, Traefik avait besoin de l'API Docker pour sa découverte dynamique. Le montage du socket Docker simplifie l'intégration, mais augmente le niveau de privilège. C'est un risque structurel que j'ai conservé dans l'analyse.

## 5. Incidents et méthode de diagnostic — 2 minutes

> Des instabilités et des pics de consommation sont ensuite apparus. Je n'ai pas considéré Traefik comme la cause unique. Sur un VPS partagé, une erreur 502 ou une latence élevée peut venir du DNS, du TLS, du proxy, du réseau overlay, de l'application, de la base, du disque ou d'une pression mémoire.

> J'ai appliqué un diagnostic vertical. D'abord, vérifier la résolution DNS et l'adresse réellement jointe. Ensuite, vérifier TCP et le pare-feu. Puis contrôler le certificat, le SNI et sa date. Après cela, inspecter la règle du reverse proxy, les middlewares et le Host. Enfin, tester l'upstream depuis le réseau overlay et examiner l'application et sa base.

> Cette méthode évite les redémarrages aveugles qui détruisent les preuves. Je collecte les horaires, les logs, l'état des tâches et les métriques avant de modifier le système. Je distingue toujours trois catégories : ce qui est observé, ce qui est une hypothèse et ce qui est confirmé par un test.

> Les comparaisons de ressources entre Traefik et Nginx restent des observations propres à cet environnement. Je ne les présente pas comme un benchmark universel. Une comparaison scientifique nécessiterait la même charge, les mêmes upstreams, les mêmes versions, une période de chauffe et des percentiles de latence.

## 6. Pourquoi passer à Nginx — 1 minute 30

> Le choix Nginx n'est pas un jugement absolu contre Traefik. Traefik reste pertinent lorsque les services apparaissent souvent et que la découverte dynamique est un besoin central. Dans notre cas, le catalogue comportait un nombre limité de routes relativement stables, l'équipe était réduite et la priorité était la lisibilité opérationnelle.

> Nginx permettait de centraliser les virtual hosts, de revoir chaque route comme du code et de supprimer l'accès de l'edge au socket Docker. En contrepartie, l'ajout d'un domaine exige une modification explicite et le cycle TLS est géré séparément par Certbot.

Phrase forte :

> **Je n'ai pas choisi l'outil le plus automatisé ; j'ai choisi le niveau d'automatisation que l'équipe pouvait expliquer, tester et reprendre.**

## 7. Architecture Nginx et cycle TLS — 3 minutes

> Dans la cible, Nginx est le seul service qui publie les ports 80 et 443. Les applications restent sur le réseau overlay et sont jointes par leur nom de service et leur port interne. Chaque virtual host applique les snippets adaptés : proxy commun, TLS, en-têtes, Basic Auth, rate limit ou CORS pour Superset.

> Sur le port 80, un serveur par défaut ferme les hôtes inconnus avec le code Nginx 444. Seuls les domaines autorisés peuvent servir le challenge ACME et être redirigés vers HTTPS. Cette règle évite de réfléchir un Host arbitraire dans une redirection ouverte.

> Pour la résolution interne, Nginx utilise le DNS Docker `127.0.0.11`. Les upstreams sont placés dans des variables afin de permettre une résolution au runtime. Il faut être précis : derrière le VIP Swarm, le remplacement d'une simple tâche ne casse pas forcément Nginx. La résolution dynamique protège surtout contre une recréation de service, un mode DNSRR ou une adresse mise en cache devenue obsolète.

> Le premier certificat crée un problème de bootstrap : Nginx ne peut pas démarrer en HTTPS avec un fichier qui n'existe pas encore. J'ai donc séparé le cycle en cinq étapes : préflight DNS, stack HTTP bootstrap, challenge Certbot en webroot, bascule vers la stack TLS, puis renouvellement périodique. Les certificats sont partagés par volume ; Nginx les monte en lecture seule.

> La configuration Swarm est immuable. Le script calcule une empreinte SHA-256 des fichiers et l'ajoute au nom des Docker Configs. Chaque changement produit donc un objet distinct. Cela rend le déploiement déterministe et conserve l'ancienne version pour le rollback.

> Avant le déploiement, le mode `validate` vérifie la syntaxe Bash, la parité entre les domaines du certificat et les routes, l'absence de tags flottants, le rendu des stacks et un véritable `nginx -t` dans l'image Nginx épinglée. Je teste l'artefact qui sera réellement exécuté, pas une version différente installée sur ma machine.

## 8. Sécurité, secrets et sauvegardes — 2 minutes

> La sécurité est organisée en défense en profondeur. L'edge limite l'exposition ; les bases n'ont plus de ports publics ; les outils administratifs ajoutent authentification et limitation ; les réseaux séparent applications et données ; les secrets sont injectés au runtime lorsque les applications le permettent.

> Je ne prétends pas que tout est parfait. Certaines stacks WordPress historiques utilisent encore des fichiers `.env`. Un secret présent dans l'environnement reste visible à travers certaines inspections et doit être migré. De même, une Basic Auth n'est pas une protection suffisante pour toutes les interfaces privilégiées : la prochaine étape est un VPN, un proxy d'identité ou une allowlist réelle.

> Pour la reprise, je distingue le dump de base, la sauvegarde des volumes et la configuration Git. Un site WordPress nécessite la base et le contenu. Passbolt nécessite la base, les clés GPG et les clés JWT. Sauvegarder une seule partie peut produire une restauration techniquement terminée mais fonctionnellement inutilisable.

> Le RPO indique la quantité de données que l'on accepte de perdre. Le RTO indique le délai de remise en service. La fréquence d'un CronJob ne prouve pas le RPO si les sauvegardes échouent. **Une sauvegarde n'est validée qu'après une restauration isolée et un test applicatif.**

## 9. Perspective K3s et observabilité — 2 minutes 45

> Le troisième dépôt prépare une évolution vers K3s, une distribution légère de Kubernetes. L'objectif est d'obtenir des APIs standardisées pour l'Ingress, les certificats, les NetworkPolicies, les comptes de service, les quotas, les probes, les CronJobs et l'observabilité.

> La plateforme sépare les applications, les données et les opérations dans des namespaces. Cette séparation n'est pas automatiquement une barrière réseau. Des politiques `default-deny` bloquent les flux, puis des règles autorisent uniquement les chemins nécessaires : Ingress vers application, application vers sa base, DNS, SMTP, Git, S3 et télémétrie.

> ingress-nginx gère l'entrée HTTP/S et cert-manager le cycle ACME. Les secrets destinés à Git sont chiffrés avec SOPS et Age. Les volumes utilisent une StorageClass locale avec rétention, mais cela ne réplique pas les données ; Restic externalise donc les dumps et les PVC vers un stockage S3 compatible.

> Pour l'observabilité, Prometheus collecte les métriques, Alertmanager route les alertes, Grafana les visualise et Loki stocke les logs. Grafana Alloy découvre les pods par l'API Kubernetes et envoie les logs à Loki. Il a remplacé Promtail, arrivé en fin de vie.

> Je présente honnêtement le niveau de preuve : les manifests se rendent avec Kustomize, les images sont épinglées et résolues, les contrôles statiques passent et la configuration Alloy est validée par son binaire. Cependant, je n'avais pas de cluster vivant dans l'espace d'audit final. K3s est donc une baseline orientée production et un laboratoire reproductible, pas une production déjà démontrée.

## 10. Résultats et feuille de route — 1 minute 45

> Le résultat principal est une réduction de l'implicite. Les domaines possèdent un catalogue, les configurations une empreinte, les images un digest, les secrets un workflow, les sauvegardes un runbook et les changements un rollback.

> Les corrections concrètes incluent la suppression des ports directs de bases de données, l'épinglage des images publiques, la sécurisation des scripts de déploiement, la protection contre les Host inconnus, la correction de la persistance GLPI, la séparation des volumes cryptographiques Passbolt et le remplacement de Promtail par Alloy.

> Les priorités restantes sont de tester les restaurations avec des données de démonstration, protéger les interfaces administratives par VPN ou SSO, migrer les derniers secrets `.env`, externaliser les logs, déployer K3s en laboratoire et décider d'une topologie multi-nœuds uniquement si le SLA et le budget le justifient.

## 11. Compétences et conclusion — 1 minute 20

> Ce projet m'a permis de développer une posture DevOps complète. J'ai travaillé sur Linux, Docker Swarm, le routage HTTP, TLS, les réseaux, les secrets, les scripts Bash, Kubernetes, l'observabilité et les sauvegardes. Surtout, j'ai appris à transformer un symptôme en hypothèses testables, à limiter le rayon d'impact d'un changement et à documenter les limites aussi précisément que les réussites.

> Pour conclure, je retiens qu'une architecture professionnelle n'est pas celle qui contient le plus d'outils. C'est celle que l'équipe peut comprendre, vérifier, faire évoluer et restaurer. Mon travail a fait évoluer la plateforme d'une configuration fonctionnelle mais hétérogène vers une infrastructure plus déterministe, mieux segmentée et accompagnée d'une trajectoire de modernisation réaliste.

Dernière phrase, puis silence :

> **La disponibilité est un état temporaire ; la résilience est une capacité démontrée.**

