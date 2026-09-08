# Questions probables du jury

Format recommandé : **réponse directe → justification → limite**. Ne pas commencer par une histoire longue.

## Architecture et décisions

### Pourquoi avoir utilisé Swarm sur un seul nœud ?

> Swarm apportait un modèle déclaratif, des services, des réseaux overlay, des secrets et un mécanisme de convergence avec un coût d'apprentissage faible. Il ne fournissait pas de haute disponibilité physique sur un seul nœud, et je le présente explicitement comme cette limite.

### Pourquoi ne pas avoir choisi Kubernetes dès le début ?

> La première priorité était de stabiliser avec l'outillage déjà maîtrisé. Introduire Kubernetes pendant une migration critique aurait augmenté simultanément la complexité de plateforme et le risque de données. K3s a été préparé ensuite comme trajectoire progressive et testable.

### Pourquoi remplacer Traefik par Nginx ?

> Le catalogue de routes était stable et l'équipe avait besoin d'une configuration centrale facile à relire. Nginx supprimait aussi le besoin du socket Docker pour l'edge. En contrepartie, l'ajout d'une route et le cycle Certbot deviennent explicites. C'est un arbitrage contextuel, pas un classement universel.

### Est-ce que Traefik était la cause des incidents ?

> Je ne peux pas attribuer tous les symptômes à Traefik. Le VPS hébergeait plusieurs workloads et les causes possibles incluaient CPU, mémoire, disque, réseau, application et base. Mon diagnostic a isolé les couches ; la migration Nginx a surtout réduit la complexité opérationnelle de l'edge.

### Pourquoi conserver Traefik dans le dépôt ?

> Le dépôt documente l'architecture initiale, ses décisions et ses risques. Il constitue une trace technique et peut rester utile pour un environnement où la découverte dynamique serait prioritaire.

### Pourquoi K3s plutôt qu'un Kubernetes complet ?

> K3s reste conforme aux APIs Kubernetes tout en réduisant l'empreinte et le nombre de composants à installer. Il est adapté à un petit laboratoire ou serveur edge. La topologie mono-nœud conserve néanmoins le même risque matériel.

## Réseau, DNS et routage

### Quelle différence entre port publié et port interne ?

> Le port interne est celui qu'écoute le conteneur sur son réseau. Un port publié ouvre un accès sur l'hôte ou le routing mesh. Le reverse proxy doit utiliser le port interne ; seules les entrées nécessaires, généralement 80 et 443, sont publiées.

### Comment Nginx trouve-t-il un service Swarm ?

> Le proxy et le service partagent un réseau overlay. Nginx interroge le DNS embarqué Docker à l'adresse 127.0.0.11 avec le nom `stack_service`. Le DNS retourne normalement un VIP ou les adresses DNSRR selon le mode.

### Un redémarrage de conteneur casse-t-il le proxy_pass ?

> Pas nécessairement. Derrière un VIP stable, Swarm remplace la tâche sans changer l'adresse du service. La résolution runtime est surtout utile lors d'une recréation du service, en DNSRR ou lorsqu'une réponse mise en cache devient obsolète.

### Pourquoi fermer les Host inconnus ?

> Une redirection fondée sur `$host` peut réfléchir une valeur contrôlée par le client. Le default server renvoie 444 et seule une allowlist de domaines connus peut rediriger vers HTTPS ou servir ACME.

### Pourquoi aucun port de base n'est publié ?

> Les applications consomment les bases sur des réseaux privés. Une publication publique augmente la surface d'attaque et contourne les contrôles de l'edge. L'administration doit passer par un canal contrôlé, par exemple VPN ou tunnel SSH.

## TLS et Nginx

### Pourquoi une stack bootstrap ?

> Le serveur HTTPS ne peut pas charger un certificat absent. La stack bootstrap sert le challenge ACME en HTTP ; après émission, la stack finale monte le certificat et active HTTPS.

### Pourquoi un certificat SAN unique ?

> Il simplifie le volume et le renouvellement pour un petit catalogue. Il couple toutefois les domaines et expose leur liste. Si les propriétaires ou cycles diffèrent, je séparerais les certificats par famille.

### Comment le certificat renouvelé est-il chargé ?

> Certbot vérifie périodiquement le renouvellement dans le volume partagé. Nginx recharge sa configuration pour rouvrir les fichiers. La version actuelle utilise un reload périodique ; un hook de renouvellement contrôlé réduirait le délai.

### Pourquoi `nginx -t` dans un conteneur ?

> Je veux tester la version et les modules réellement déployés. Un Nginx installé localement pourrait accepter ou refuser une directive différemment.

### Pourquoi des snippets ?

> Ils centralisent les politiques communes et évitent la duplication. Les exceptions, comme l'intégration iframe de Superset, restent dans un profil séparé pour ne pas affaiblir toutes les routes.

## Sécurité

### Basic Auth est-elle suffisante ?

> C'est une barrière complémentaire, pas la protection finale d'une interface privilégiée. La cible recommandée ajoute VPN, proxy d'identité ou allowlist réelle, avec TLS et authentification applicative.

### Pourquoi le socket Docker est-il sensible ?

> L'API Docker permet des opérations proches d'un contrôle root du nœud : création de conteneurs privilégiés, montages et accès aux secrets. Traefik en a besoin pour la découverte ; Nginx statique non.

### Les Docker Secrets sont-ils chiffrés ?

> Swarm protège les secrets dans l'état Raft du manager et les distribue aux tâches autorisées. Au runtime, ils apparaissent comme fichiers. Cela réduit l'exposition mais n'empêche pas l'application de les divulguer dans ses logs.

### Base64 protège-t-il un Secret Kubernetes ?

> Non. Base64 est un encodage. La protection repose sur RBAC, chiffrement au repos, contrôle du kubeconfig et, dans Git, un outil comme SOPS avec Age.

### Pourquoi épingler les images par digest ?

> Un tag peut changer sans modification du manifeste. Le digest fixe le contenu exact et rend le rollback reproductible. Il ne remplace pas le scan de vulnérabilités ni la mise à jour régulière.

### Qu'est-ce que le moindre privilège ?

> Chaque workload reçoit uniquement les accès nécessaires. Une app n'accède qu'à sa base, un pod sans besoin API ne monte pas de token, Alloy lit les pods mais ne modifie pas les workloads.

## Sauvegarde et résilience

### Pourquoi un volume n'est-il pas une sauvegarde ?

> Il protège contre le remplacement du conteneur, pas contre la perte du serveur, la corruption, la suppression logique ou un attaquant ayant les mêmes droits. La sauvegarde doit être chiffrée, externalisée et restaurable.

### Différence entre RPO et RTO ?

> Le RPO mesure la perte de données acceptable ; le RTO mesure le délai de reprise visé. Ils doivent être définis par service et prouvés par un exercice.

### Que faut-il sauvegarder pour Passbolt ?

> La base MariaDB, les clés GPG et les clés JWT. Restaurer uniquement la base peut laisser les données indéchiffrables ou rendre le service incohérent.

### Comment prouvez-vous qu'une sauvegarde fonctionne ?

> Je la restaure dans un environnement isolé, je vérifie l'intégrité technique, puis j'exécute un parcours fonctionnel. Je mesure le temps et l'écart de données pour comparer RTO et RPO.

## K3s et observabilité

### Un namespace isole-t-il le réseau ?

> Non. Il organise et porte des politiques, mais il faut une NetworkPolicy et un CNI qui l'applique. Je vérifie ensuite un flux autorisé et un flux interdit.

### Pourquoi Prometheus et Loki ?

> Prometheus traite les métriques temporelles ; Loki traite les logs. Grafana interroge les deux. Les séparer permet d'utiliser un modèle adapté à chaque signal.

### Pourquoi remplacer Promtail ?

> Promtail a atteint sa fin de vie. Grafana Alloy est la trajectoire soutenue et permet ici une découverte des pods par l'API Kubernetes sans monter directement le répertoire de logs de l'hôte.

### K3s est-il prêt pour la production ?

> Le dépôt est orienté production et validé statiquement, mais il reste un laboratoire. Il manque les preuves en cluster : CNI, Ingress, ACME, alerting, charge et restaurations réelles.

### Pourquoi Portainer n'a-t-il pas un ClusterRole admin ?

> Le baseline n'accorde pas un privilège global sans besoin approuvé. Portainer reste une interface optionnelle. Si la gestion Kubernetes devient nécessaire, je définirai un rôle limité à des opérations documentées.

## Méthode et posture

### Quelle est votre contribution personnelle ?

> J'ai réalisé l'inventaire, l'analyse de capacité, la conception des architectures, les manifests, les scripts de validation, le durcissement, les runbooks et la trajectoire K3s. Mon apport principal est d'avoir transformé une configuration fonctionnelle en système explicable et contrôlable.

### Quel est votre principal apprentissage ?

> Ne pas confondre disponibilité instantanée et résilience. Un service peut répondre aujourd'hui sans que ses secrets, sa restauration ou son rollback soient maîtrisés.

### Que feriez-vous avec un mois supplémentaire ?

> Je commencerais par les preuves manquantes : restore drills, VPN/SSO pour les interfaces admin, CI complète, export des logs et cluster K3s de laboratoire. Je n'ajouterais pas de nouvelle technologie avant d'avoir fermé ces risques.

### Les pull requests prouvent-elles la qualité de votre travail ?

> Elles montrent une démarche d'apprentissage et de revue open source. Leur statut n'est pas une preuve de validation de l'infrastructure ; mes preuves sont les tests, les manifests et les procédures reproductibles.

