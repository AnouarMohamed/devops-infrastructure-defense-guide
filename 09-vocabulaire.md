# Vocabulaire et phrases fortes

## Termes à utiliser correctement

| Terme | Signification dans le projet | Phrase prête à l'emploi |
|---|---|---|
| État désiré | Déclaration que l'orchestrateur essaie de rendre vraie. | « Swarm fait converger l'état réel vers l'état désiré. » |
| Déterminisme | Même entrée, résultat identifiable et prévisible. | « Les digests et empreintes réduisent la dérive entre validation et déploiement. » |
| Idempotence | Une opération peut être relancée sans destruction préalable. | « Le script converge vers la stack demandée au lieu de la supprimer. » |
| Immutabilité | Un artefact validé n'est pas modifié en place. | « Chaque contenu Nginx crée une nouvelle Docker Config. » |
| Moindre privilège | Accès minimal nécessaire à la fonction. | « Le proxy statique n'a pas besoin du socket Docker. » |
| Défense en profondeur | Plusieurs contrôles complémentaires. | « Je combine exposition minimale, segmentation, secrets, logs et reprise. » |
| Surface d'attaque | Ensemble des chemins exploitables. | « Retirer les ports DB publics réduit directement la surface d'attaque. » |
| Frontière de confiance | Passage entre zones de niveaux de confiance différents. | « L'edge est la frontière entre Internet et les services overlay. » |
| Rayon d'impact | Étendue affectée par une erreur ou un changement. | « Les snippets ciblés limitent le rayon d'impact d'une exception CORS. » |
| Observabilité | Capacité à expliquer l'état à partir de signaux. | « Je corrèle métriques, logs, état des tâches et événements. » |
| Réversibilité | Capacité à revenir à l'état précédent. | « La migration conserve l'ancien edge et les configs versionnées. » |
| Dette technique | Risque ou travail connu, non encore fermé. | « Les secrets `.env` historiques sont une dette explicitement priorisée. » |
| RPO | Perte de données maximale acceptable. | « Le RPO doit être défini par criticité métier. » |
| RTO | Délai cible de remise en service. | « Le restore drill mesure le RTO réel. » |
| Source de vérité | Référence principale qui doit rester cohérente. | « Le catalogue de domaines est la source de vérité du certificat. » |

## Phrases fortes

- « Je sépare ce qui est observé, ce qui est supposé et ce qui est prouvé. »
- « Une automatisation utile retire une erreur fréquente sans masquer la défaillance. »
- « Le choix technique est proportionné au besoin et à la capacité d'exploitation. »
- « Je réduis les chemins publics au lieu d'ajouter des contrôles sur des chemins inutiles. »
- « Le manifeste déclare l'intention ; le test runtime prouve le comportement. »
- « Le digest garantit l'identité de l'image, pas sa sécurité absolue. »
- « Un dashboard facilite l'analyse ; il ne remplace pas une preuve reproductible. »
- « La sauvegarde protège les données seulement si sa restauration est démontrée. »
- « Un orchestrateur mono-nœud apporte de la convergence, pas de la redondance matérielle. »
- « Nginx était mieux aligné avec le contexte ; Traefik n'était pas intrinsèquement mauvais. »
- « Je prépare le rollback avant le changement, pas après l'incident. »
- « La résilience est une capacité mesurée, pas une propriété déclarée. »

## Formulations faibles à remplacer

| Éviter | Dire |
|---|---|
| « Traefik consomme trop. » | « Des écarts ont été observés dans cette charge ; ils ne constituent pas un benchmark universel. » |
| « Nginx est meilleur. » | « Nginx correspondait mieux à un catalogue stable et au modèle d'exploitation. » |
| « Swarm assure la haute disponibilité. » | « Swarm assure la convergence ; le nœud unique reste un point de défaillance. » |
| « Les secrets sont sécurisés. » | « Les secrets sont moins exposés et leur rotation reste nécessaire. » |
| « K3s est production-ready. » | « La baseline est orientée production ; les preuves runtime restent à obtenir. » |
| « J'ai tout automatisé. » | « J'ai automatisé les contrôles déterministes et conservé une validation humaine sur les mutations risquées. » |
| « La sauvegarde marche. » | « La sauvegarde a été créée ; elle ne sera validée qu'après un restore drill. » |
| « J'ai corrigé le serveur. » | « J'ai réduit des risques identifiés et documenté les risques résiduels. » |

## Répondre quand on ne sait pas

Ne pas inventer.

> « Je n'ai pas validé ce point dans un environnement réel. Mon hypothèse est X, et je la vérifierais avec Y avant de prendre une décision. »

Cette réponse montre méthode et honnêteté.

